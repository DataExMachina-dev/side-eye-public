To see the docs, run
```shell
hugo server --theme=hugo-book
```
Make sure you've run `git submodule update --init`.

To deploy a new version of the docs to the S3 bucket, make sure you've done 
`aws sso login` and then:
```shell
hugo deploy --target production
```
