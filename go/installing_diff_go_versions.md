# Issue with installing different go versions using asdf

I use asdf to manage different golang versions.
Unfortunately many times I have run into issues. More specifically when trying to change versions of go using:

```
asdf local golang 1.20.10
go version
go version go1.19.3 darwin/arm64
```

This can happen for a couple of reasons. First check your `go env` and pay attention to `GOPATH` and `GOROOT`.

These variables should change when executing `asdf local golang <version>`
Assuming that you are using `zsh` open your `~/.zshrc` file and add the following:

```
export GOPATH=$(asdf where golang)
export PATH="${PATH}:$(go env GOPATH)/go/bin/go"
```

This will make sure that both `GOPATH` and `PATH` have the correct value.
Also make sure that when executing `asdf where golang` you don't get a warning. If you do get a warning consider adding:

```
export ASDF_GOLANG_MOD_VERSION_ENABLED=true
```

before the previous commands.

Remember that you need to restart your terminal to see the effect in the `go env`.

## Visual Studio Code still points to the old golang version

On the bottom right corner (blue line) you can see which go version vs code is currently using.
Click on it and choose change environment. If this doesn't work follow this [guide](https://stackoverflow.com/questions/76306692/switching-go-version-when-process-envgoroot-is-set-is-unsupported). Don't forget to restart VS code. You need to close **all** VS code windows to really restart it.

If the `go version` is not there just open any go.mod file and it should appear. 








