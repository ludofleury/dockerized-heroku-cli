# Heroku dockerized CLI

If you like to isolate your toolbelt into docker container, here's a quick hack to avoid installing locally the Heroku CLI

## Let's go

First manually clone the CLI (update are your responsibility)
```
  git clone https://github.com/heroku/cli /your/heroku/cli
```

Actually, the repository already provides a Dockerfile, not sure why they didn't publish an official image
```
  cd /your/heroku/cli
  docker build -t heroku-cli .
```

You can run the heroku cmd
```
  docker run --rm -it heroku-cli
```

Yet we should run the heroku cmd like the following
```
  docker run --rm -it -v /your/shared/host-path:/root heroku-cli heroku --version
```

Why? Because heroku relies on an Oauth2 token (any heroku cmd will re-use them)
In order to follow the docker philosophy, we will mount a volume from your host (you machine) into the heroku cli container.
This way, it will not pollute the container AND be re-used from your host every time, cool!
Let's login as instructed by the heroku documentation, the dockerized way
```
  docker run --rm -it -v /your/shared/host-path:/root heroku-cli heroku login -i
```

Finally, we add a little trick to also mount the current directory (your git repository) as the current working dir
```
  docker run --rm -it -v /your/shared/host-path:/root -v `pwd`:/var/app -w="/var/app" heroku-cli heroku [cmd]
```

## And what about git push heroku master?

We do not need to use the containerized cli for the `git push heroku branch` because heroku cli just add a remote host to your current local git repository
If you added your ssh-keys to heroku, the push should work out-of-the-box
git is one of the core tool that I do not recommend to dockerize

## Adding SSH key with the dockerized cli
We have to mount your ssh keys as a volume into the container as well just for that process.
Following the [heroku documentation about SSH keys](https://devcenter.heroku.com/articles/keys), we just hack a bit the command:

```
  docker run --rm -it -v /your/shared/host-path:/root -v /your/path/to/ssh:/root/.ssh -w="/var/app" heroku-cli heroku keys:add
```

## Keyboard Laziness

Tired to type the whole cmd everytime?
Add an alias to your `.bashrc` or `.zshrc` etc

```
  alias heroku="docker run --rm -it -v /your/shared/host-path:/root -v `pwd`:/var/app -w="/var/app" heroku-cli heroku"
```

Done. What happens? you can now use heroku directly, and it will actually:
- run an ephemeral docker container with the cli
- re-use Oauth2 credentials from your host
- mount the current directory as the working dir for the cli


If you always navigate to your project folder/repository, everything must work like the official documentation states it
