# abp-single-lambda-api

[aws-blueprint](https://github.com/rynop/aws-blueprint) example for an API backed by a single lambda, using **golang**

## Setup

Follow [setup steps](https://github.com/rynop/validate-abp-single-lambda-api#setup) that are common across all languages

### CI/CD CloudFormation Parameters:

*  `TestBuildPackageSpecPath`: `aws/codebuild/go-test-package.yaml`
*  `LambdaPublishBuildSpecPath`: `aws/codebuild/lambda-publish.yaml`
*  `HandlerPath`: `cmd/apighandler/main`

Project layout is based on [golang-standards/project-layout](https://github.com/golang-standards/project-layout)

We recommend using [retool](https://github.com/twitchtv/retool) to manage your tools (like (dep)[https://github.com/golang/dep]).  Why?  If you work with anyone else on your project, and they have different versions of their tools, everything turns to shit.

1. (Install retool)[https://github.com/twitchtv/retool#usage]: `go get github.com/twitchtv/retool`. Make sure to add `$GOPATH/bin` to your PATH
1. These commands should be run in each of your go projects
    1. `retool add github.com/golang/dep/cmd/dep origin/master`
    1. `retool add github.com/golang/lint/golint origin/master`
    1. `retool do dep init`.  In this case, since we already have a `dep` project setup, run `retool do dep ensure`
1. Add dependency example: `retool do dep ensure -add github.com/apex/gateway github.com/aws/aws-lambda-go`

## Twirp Lambda

Want to use a [Twirp RPC framework](https://github.com/twitchtv/twirp) based lambda instead?

### CI/CD CloudFormation Parameters:

*  `TestBuildPackageSpecPath`: `aws/codebuild/go-test-package.yaml`
*  `LambdaPublishBuildSpecPath`: `aws/codebuild/lambda-publish.yaml`
*  `HandlerPath`: `cmd/apighandlertwirp/main`

Same project layout layout, `retool` and `dep` steps as above.

1.  Install plugins locally:
    1.  `retool add github.com/golang/protobuf/protoc-gen-go origin/master`
    1.  `retool add github.com/twitchtv/twirp/protoc-gen-twirp origin/v6_prerelease`
1.  Auto-generate the code:
```
retool do protoc --proto_path=$GOPATH/src:. --twirp_out=. --go_out=. ./rpc/publicservices/service.proto 
retool do protoc --proto_path=$GOPATH/src:. --twirp_out=. --go_out=. ./rpc/adminservices/service.proto 
```    
1. For this example, the interface implementations have been hand created in `pkg/`. Take a look.
1. Example to consume twirp API in this example: `curl -H 'Content-Type:application/json' -H 'Authorization: Bearer aaa' -H 'X-FROM-CDN: <your VerifyFromCfHeaderVal>' -d '{"term":"wahooo"}' https://<--r output CNAME>/com.rynop.twirpl.publicservices.Image/CreateGiphy`

Testing locally:
1.  Set `LOCAL_LISTEN_PORT` and `X_FROM_CDN` env vars. (Fish: `set -gx LOCAL_LISTEN_PORT 8080`, `set -gx X_FROM_CDN localTest`)
1.  Build & run: `cd cmd/apighandlertwirp`, `go build -o /tmp/main .; /tmp/main`
1.  Hit endpoint: `curl -v -H 'Content-Type:application/json' -H 'Authorization: Bearer aaa' -H 'X-FROM-CDN: localTest' -d '{"term":"wahooo"}' http://localhost:8080/com.rynop.twirpl.publicservices.Image/CreateGiphy`
