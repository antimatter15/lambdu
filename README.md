<img src="https://raw.githubusercontent.com/antimatter15/lambdu/master/images/logo.png" alt="eigensheep" width="500"/>

![PyPI](https://img.shields.io/pypi/v/eigensheep.svg)
![PyPI - Python Version](https://img.shields.io/pypi/pyversions/eigensheep.svg)
![PyPI - License](https://img.shields.io/pypi/l/eigensheep.svg)

Eigensheep is a python package, with a [very easy setup process](#getting-started), that lets you effortlessly run Jupyter Notebook cells on AWS Lambda, enabling massive parallelism. 
To instantly provision and run your code on 1000 tiny VMs, prefix a cell with `%%eigensheep -n 1000`. 

Eigensheep gives your Lambda code full access both to packages from PyPi, and to layers from [Lambda Layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html), including typically tricky-to-install things like `Z3`, `ffmpeg`, and `puppeteer`.

## Features

- Just prefix a cell with `%%eigensheep` to run it on AWS Lambda
- Automatically generates Lambda deployment packages with pre-installed dependencies via `pip`. 
- Supports Lambda Layers for easily including external libraries like Z3, FFmpeg, Puppeteer/Chromium, LibreOffice, Tesseract OCR, YOLOv3 on Darknet, and Spacy
- Automatically caches Lambda configurations
- Supports response sizes over 6MB by saving results to S3
- Integrates `tqdm` for interactively displaying progress
- Easy setup and configuration powered by AWS CloudFormation 
- Automatically copies variables from notebook scope


<img src="https://raw.githubusercontent.com/antimatter15/lambdu/master/images/chart.png" alt="Sequentially opening 50 websites with Puppeteer and taking screenshots takes 105.6 seconds, while the same task split into 50 concurrent Lambda invocations finishes in 9.8 seconds" width="500"  />

Here we compare the task of capturing screenshots of the 50 most popular websites (according to Moz) with Pyppeteer. In the first bar, we're doing this sequentially with a Python `for` loop. In the second one, each website is run as a different Lambda. The estimated cost of the full sequential test (at the current `us-east-1` price) is $0.0051 (or 0.07% of the monthly free quota). The estimated cost of the full parallel test is $0.0073 (or 0.11% of the monthly free quota).

## Getting Started

Open up your Terminal and install `eigensheep` with `pip`

    pip install eigensheep

Open a Jupyter notebook with `jupyter notebook` and create a new Python notebook. Eigensheep supports both Python 2 and Python 3. Run the following code in a cell:

    import eigensheep

Follow the on-screen instructions to configure AWS credentials. AWS credentials will be saved to `~/.aws/config` under the `eigensheep` profile for subsequent invocations. Eigensheep uses AWS CloudFormation so you only need a few clicks to get started (see our [guided video walkthrough](https://www.youtube.com/watch?v=jdurk2DRdAs)). 

[<img src="https://raw.githubusercontent.com/antimatter15/lambdu/master/images/setup.png" alt="eigensheep setup" width="500" />](https://www.youtube.com/watch?v=jdurk2DRdAs)

Once Eigensheep is set up, you can run any code on Lambda by prefixing the cell with `%%eigensheep`. You can include dependencies from `pip` by typing `%%eigensheep <list of package names>`, for example `%%eigensheep requests numpy`. You can invoke a cell multiple times concurrently with the `-n` parameter, for example `%%eigensheep -n 100`. 

[<img src="https://raw.githubusercontent.com/antimatter15/lambdu/master/images/puppet.gif" alt="eigensheep usage" width="500"  />](https://www.youtube.com/watch?v=jdurk2DRdAs)

## Frequently Asked Questions

**Q: Why is this library called Eigensheep?**

The name comes from the classic math joke:

> What do you call a baby eigensheep? 
> 
> A [lamb, duh](https://en.wikipedia.org/wiki/Eigenvalues_and_eigenvectors#Overview). 

**Q: Does this work on Python 2 and Python 3?**

A: Both Python 2 and Python 3 are supported. If the library is imported from a Python 2.x notebook, the Lambda runtime will default to "python2.6". If the library is imported from a Python 3.x notebook, the Lambda runtime defaults to "python3.6". This can be manually overridden with the "--runtime" option.

**Q: Can I use this to do GPU stuff?**

A: Currently the AWS Lambda execution environment does not expose access to any GPU acceleration. Eigensheep probably won't be that useful for training deep neural nets.

**Q: How much does it cost to run stuff on AWS Lambda?**

A: Unlike a traditional VM, you don't get charged while you're idling and not actively computing. You don't have to worry about accidentally forgetting to turn off a machine, and provisioning a VM takes only milliseconds rather than minutes. 

AWS provides a pretty generous Free Tier for Lambda which does not expire after 12 months. It's 400,000 GB-seconds/month. That's 36 continuous hours of a single maxed out 3108MB Lambda job for free every month. Alternatively, it's about 20 minutes of 100 concurrent maxed out instances. After that it's about $7 for every subsequent free-tier equivalent. 

**Q: Can this be used for web scraping?**

A: Yes, Eigensheep can be used for web scraping. However, note that different Lambda VM instances often share the same IP address. 

**Q: Can Eigensheep be used for long running computations?**

A: The maximum allowed duration of any Lambda job is 15 minutes. Eigensheep works best for tasks which can be broken up into smaller chunks. 


**Q: What are the security implications of using Eigensheep?**

A: The Eigensheep CloudFormation stack creates an IAM User, Access Key, and Lambda Role with as few permissions as possible. If the access keys are compromised, the attacker only has access to a bucket containing Eigensheep-specific content, and can not use it to access any of your other AWS resources. 

The IAM User can only read/write from a specific bucket earmarked for use with Eigensheep, and can only update a specific lambda function (all the different variants are stored as different versions on a single Lambda function). The Lambda function only has access to the specific bucket and the ability to write to CloudWatch logs and XRay tracing streams. 

All of the access keys can be revoked and all of the resources can be removed simply by deleting the CloudFormation stack from the AWS console. 


**Q: Where does Eigensheep store its configuration?**

A: Eigensheep stores its access keys and configuration in the `~/.aws/config` file under the `eigensheep` profile.


**Q: Can I use Eigensheep without installing the CloudFormation Stack?**

A: Yes. Although it's a bit more complicated to set up. You can use any AWS access key and secret, so long as it has the ability to modify/invoke a Lambda named "EigensheepLambda" (which must be manually created). You must also create an S3 bucket named "eigensheep-YOUR_ACCOUNT_ID", where YOUR_ACCOUNT_ID is your numerical AWS account ID.

## Usage

```
usage: %%eigensheep [-h] [-n N] [--memory MEMORY] [--timeout TIMEOUT]
                    [--runtime RUNTIME] [--layer LAYER] [--reinstall]
                    [--no_install] [--clean] [--rm] [--name NAME] [--verbose]
                    [deps [deps ...]]

Jupyter cell magic to invoke cell on AWS Lambda

positional arguments:
  deps               dependencies to be installed via pip

optional arguments:
  -h, --help         show this help message and exit
  -n N               number of parallel lambdas to invoke
  --memory MEMORY    amount of memory in 64MB increments from 128 up to 3008
  --timeout TIMEOUT  lambda execution timeout in seconds up to 900 (15
                     minutes)
  --runtime RUNTIME  lambda runtime (python3.7, python2.7) defaults configured
                     based on host environment
  --layer LAYER      ARNs of lambda layers to include
  --reinstall        regenerate lambda configuration and dependencies
  --no_install       do not install dependencies if configration not found
  --clean            clear all deployed lambda configurations
  --rm               remove a specific lambda configuration
  --name NAME        store the lambda for later use with `eigensheep.map` or
                     `eigensheep.invoke`
  --verbose          show additional information from lambda invocation
```


`eigensheep.map("do_stuff", [1, 2, 3, 4])`


`eigensheep.invoke("do_stuff")`



```
%eigensheep --clean
```


## Acknowledgements

This library was written by [Kevin Kwok](https://twitter.com/antimatter15) and [Guillermo Webster](https://twitter.com/biject). It is based on Jupyter/IPython, `tqdm`, `boto3`, and countless Stackoverflow answers.

If you're interested in this project, you should also check out [PyWren](http://pywren.io/) by Eric Jonas, and [ExCamera](https://www.usenix.org/system/files/conference/nsdi17/nsdi17-fouladi.pdf) from Sadjad Fouladi, et al. 
