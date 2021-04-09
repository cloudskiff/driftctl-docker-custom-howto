# How to build your own driftctl Docker image (including your own .driftignore file)

The easiest way to use driftctl along with your own custom `.driftignore` file is to build your own docker container image, based on [the official one](https://hub.docker.com/repository/docker/cloudskiff/driftctl).

Requirements for this tutorial:

- an AWS IAM Keypair for driftctl (read more [in the documentation for the least priviledged policy](https://docs.driftctl.com/0.7.0/providers/aws/authentication))
- your Terraform states on S3 (read more [in the docs how to access them](https://docs.driftctl.com/0.7.0/usage/cmd/scan-usage#--from))
- Docker or similar

Here´s how we do it:

- First confirm that you can run `driftctl` from [the official docker image](https://hub.docker.com/repository/docker/cloudskiff/driftctl):

```shell
$ docker run -t --rm -v $(pwd):/app:ro \
  -e AWS_ACCESS_KEY_ID=AKIAxxx \
  -e AWS_SECRET_ACCESS_KEY=XXX \
  -e AWS_REGION=us-east-1 \
  cloudskiff/driftctl scan --from tfstate+s3://mycorp-bucket/tfstates-folder/ 
[...]
```

- Now  create a new folder (that will eventually become a git repository later) and add your own `.driftignore` file to it:

```shell
mkdir mycorp-driftctl-docker-custom 
touch .driftignore 
```

- Add all the content you need into this `.driftignore` file, like:

```shell
echo "aws_iam_user.terraform" >> .driftignore 
[...lots of copy-pasting...]
```

- Now, let's create our own Docker image from the official one; create a `Dockerfile` and open it:

```shell
touch Dockerfile 
```

- Add the following content:  

```Dockerfile
FROM cloudskiff/driftctl 
WORKDIR /app 
COPY .driftignore . 
```

- Now build your custom docker image:  

```shell
$ docker build -t mycorp-driftctl . 
[...] 
```

- Finally run your own docker image of `driftctl`:  

```shell
$ docker run -t --rm \
  -e AWS_ACCESS_KEY_ID=AKIAxxx \
  -e AWS_SECRET_ACCESS_KEY=xxx \
  -e AWS_REGION=us-east-1 \
  mycorp-driftctl:latest scan --from tfstate+s3://driftctl-tfstates/ 
[...]
```

Congratulations! Now you can run `driftctl` fully isolated in Docker, with your own `.driftignore` always available. Much easier now to run as a scheduled task or cronjob!  

## Next Steps

The next steps can include:  

- Adding this folder to git, so you can version the `.driftignore` file and track your improvements over time
- Pushing the image somewhere central, so it can be easily used by your team
- Automatically rebuilding your own docker image when the source image changes (so you stay up to date with driftctl)
- Automatically rebuilding your own docker image when the `.driftignore` file changes
- Adding your docker image scan to an hourly cron job so you are notified when something drifts

We'd love to hear about your own use cases, come tell us how you use driftctl! Thanks for reading!
