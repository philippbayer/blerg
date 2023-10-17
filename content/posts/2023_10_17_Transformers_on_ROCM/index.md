---
title: "Getting HuggingFace Transformers to work on AMD GPUs with ROCm"
date: 2023-10-17
description: i ramble about ROCm
---

AMD GPUs are being rolled out in high-performance clusters across the world, including the new Setonix cluster at the Pawsey Supercomputing Centre in Perth. AMD GPUs don't use CUDA, they use ROCm, and let's say ROCm has not received as much attention as CUDA: not everything runs straight out of the box.

Here's how I got ROCm to work with 🤗 HuggingFace Transformers on Setonix.

First, I use this alias for nicer work on the `gpu-dev` queue:

    alias getgpunode='salloc -p gpu-dev --nodes=1 --gpus-per-node=1 --account=${PAWSEY_PROJECT}-gpu'   

Notice the -gpu at the end of the account: currently Pawsey treats GPU jobs on a different account from your 'main' account.

I am using conda (mamba) on /scratch here. Pawsey staff doesn't seem to be the biggest fan of conda as it makes many, many files per environment which slows down /scratch for everyone. It's better to use Docker images. But with machine learning/deep learning libraries especially with models stored on 🤗 Hugging Face spaces I've always had to fight to get just the right 'mix' of package version numbers to make the model work: it's easier to do that in conda first, then to build the Docker image from there.

I'm making a fresh conda environment somewhere on /scratch: (/software can also be used, but file limits are strict: check using `lfs quota /software`). The 30-day deletion policy will eventually get rid of this environment.

    mamba create -p (somewhere on scratch)/transformers transformers python=3.10
    conda activate (somewhere on scratch)/transformers

Now we need to install the right version of PyTorch. There are CUDA-specific versions of PyTorch so there are ROCm-specific version of PyTorch. At the time of this writing, Pawsey has two ROCm versions as modules:

The following code is run in an interactive GPU session via the above `getgpunode` alias:

    module avail rocm

tells us there's 5.2.3 (default, *D*) and 5.4.3. Let's load 5.4.3 because we live in the future:

    module load rocm/5.4.3

We need to install a PyTorch version which is as close as possible to 5.4.3. I can only find 5.4.2 so let's use that:

    pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/rocm5.4.2 --cache-dir (somewhere on scratch)/pipcache

Notice that I'm setting --cache-dir to somewhere on /scratch, too, as the default is in your home-directory and the file limits there are even stricter.

We can test whether PyTorch can see the GPU via ROCM:

    srun python -c 'import torch; print(torch.cuda.is_available())'

This should print `True`.

Now we can install the right tensorflow for ROCm:

    pip install tensorflow-rocm

Again, just like pip, Transformers puts its huge models into your home-directory leading to all kinds of errors as it hits the file number limits.

    export TRANSFORMERS_CACHE=(somewhere on scratch)/tf_cache

For Transformers to work I also had to upgrade `accelerate`:

    pip install -U accelerate

Now we should have everything in place to run Transformers. IMHO, the installation process wasn't *too* bad, similar in pain to installing this stuff under CUDA. There are still some models on  🤗HuggingFace that I can't get to run under CUDA... It mostly boils down to using the correct index-url for torch and the right flavour of tensorflow.

I was able to run the snippet of code on the bottom of the page for InstaDeepAI's The Nucleotide Transformer model [nucleotide-transformer-500m-1000g](https://huggingface.co/InstaDeepAI/nucleotide-transformer-500m-1000g), here's my code:

	import torch
	print(torch.cuda.is_available())
	import sys

	from transformers import AutoTokenizer
	from datasets import Dataset
	from transformers import DataCollatorWithPadding
	from transformers import TrainingArguments, Trainer, logging
	from transformers import AutoModelForSequenceClassification

	import pandas as pd
	import numpy as np
	from sklearn.metrics import f1_score, accuracy_score
	import logging
	from sklearn.model_selection import train_test_split

	def import_data(csv_file):
		"""
	    Import the csv file and get it ready for use. Ensures each file gets the same treatment.

	    in -> csv_file - string representing the location of the csv file
	    out -> pandas dataframe
	    """
	    df = pd.read_csv(csv_file)
	    df.rename(columns = {'sequence': 'text'}, inplace = True)

	    return df

	def preprocess_function(examples):
		"""
	    Tokenize the text to create input and attention data

	    in -> dataset (columns = text, label)
	    out -> tokenized dataset (columns = text, label, input, attention)
		"""
	    return tokenizer(examples["text"], truncation=True)

	def pipeline(dataframe):
		"""
	    Prepares the dataframe so that it can be given to the transformer model

	    in -> pandas dataframe
	    out -> tokenized dataset (columns = text, label, input, attention)
		"""
	    # This step isn't mentioned anywhere but is vital as Transformers library only seems to work with this Dataset data type
	    dataset = Dataset.from_pandas(dataframe, preserve_index=False)
	    tokenized_ds = dataset.map(preprocess_function, batched=True)
	    tokenized_ds = tokenized_ds.remove_columns('text')
	    return tokenized_ds
	f = sys.argv[1]
	# Load train
	train_val_df = import_data(f)

	train_df, val_df = train_test_split(train_val_df[['text', 'label']],
	                                   test_size = 0.2, random_state = 42)
	tokenizer = AutoTokenizer.from_pretrained("InstaDeepAI/nucleotide-transformer-500m-1000g",
						    num_labels = len(set(train_val_df['label'])))
	tokenized_train = pipeline(train_val_df)
	tokenized_val = pipeline(train_val_df)
	model = AutoModelForSequenceClassification.from_pretrained("InstaDeepAI/nucleotide-transformer-500m-1000g",
								    trust_remote_code = True,
								    num_labels=len(set(train_val_df['label'])))

	data_collator = DataCollatorWithPadding(tokenizer=tokenizer)

	training_args = TrainingArguments(
	    output_dir=f"./results_{sys.argv[1].replace('.csv', '')}",
	    save_strategy = 'epoch',
	    optim="adamw_torch",
	    resume_from_checkpoint = True,
	    learning_rate=2e-5,
	    per_device_train_batch_size=32, # fiddle with this until it crashes
	    per_device_eval_batch_size=32,
	    save_total_limit = 2,
	    fp16 = True,
	    num_train_epochs=70,
	    weight_decay=0.01,
	    logging_steps = 20,
	    report_to="none", 
	)

	trainer = Trainer(
	    model=model,
	    args=training_args,
	    train_dataset=tokenized_train,
	    eval_dataset=tokenized_val,
	    tokenizer=tokenizer,
	    data_collator=data_collator,
	)

	trainer.train(resume_from_checkpoint=True)

	trainer.save_model(sys.argv[1] + '_' + 'MODELSAVES')

This script will read in a CSV of training data and write a trained model into a folder ending in MODELSAVES. The data looks like this:

    sequence,label
    ACCGCGGTTATACGAGAGGCCCAAGTTGATAGACTCCGGCGTAAAGAGTGGTTAAGATAAATTTTAAACTAAAGCCGAACGCCCTCAAAGCTGTTATACGC,0

After training, you can load the model and do some predictions:

    model = AutoModelForSequenceClassification.from_pretrained(FOLDER_WE_SAVED_TO)
    trainer = Trainer(model=model, tokenizer = tokenizer, data_collator=data_collator)
    preds = trainer.predict(YOUR_DATA)
    preds_flat = [np.argmax(x) for x in preds[0]]
    print(preds_flat[:5])

How do you know that it's actually using the GPU? Here's some Python code that'll tell you:

    print(f'devices: {torch.cuda.device_count()}')
    print(f'current dev: {torch.cuda.current_device()}')
    print(f'current dev name: {torch.cuda.get_device_name(torch.cuda.current_device())}')

This prints out whether Python sees the GPU in the first place. On Setonix, you can also ssh into the nodes that are running your jobs! Use `squeue` to see which node runs your job. After ssh-ing in, run `rocm-smi` to see the GPU usage on the current node.


Now we're ready to train big, big models on a cluster with 192 GPU nodes, each with effectively eight GPUs (4 GPUs with 2 'Graphics Complex Dies' (GCDs)). Happy training!

P.S.: Next we'll have to package the above up in a Dockerfile - that's for another day. No need to waste all that space on thousands of conda-generated files when we can just have one neat Docker-image.

P.P.S.: See the [Pawsey documentation for more detailed info](https://support.pawsey.org.au/documentation/display/US/Setonix+GPU+Partition+Quick+Start), especially some useful SLURM scripts and ways to run several GPUs at once.

P.P.P.S.: Some errors I encountered:

`python3.10/site-packages/torch/lib/libtorch_cpu.so: undefined symbol: roctracer_next_record, version ROCTRACER_4.1`: this happened when I installed the wrong PyTorch version for the wrong ROCm. Make sure the index-url's ROCm version is close to the ROCm offered by Setonix, and that you're loading the right module version.

`Device-side assertion t >= 0 && t < n_classes' failed.` - in my training data, I started my class labels with a random large number instead of zero. I replaced class-labels all by a counter starting from 0.

