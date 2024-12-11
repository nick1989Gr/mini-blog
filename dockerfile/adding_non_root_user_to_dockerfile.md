# How to add a non-root user to a dockerfile

Many times we have to deal with a vulnerability like :
`A user should be specified in the dockerfile, otherwise the image will run as root`

In order to avoid that we can easily add a user to the docker file.
Be aware that if your image tries to write to a certain path in the docker file system then we need to give it the right permissions.

For example let's take an application that was running as root in the `/app/` dir and was writting files in that directory.

We can add the following code snippet

```Dockerfile
# create myuser user and give write access to app dir
RUN adduser --disabled-password myuser && \
    chown gitlab:root /app/ && \
    chmod g+w /app/

USER gitlab
WORKDIR /home/gitlab
ENV PATH /home/gitlab:${PATH}
```
