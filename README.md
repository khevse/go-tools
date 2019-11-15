# Golang tools

## hub.docker.com

Push to https://hub.docker.com/

```bash
docker login -u <login>
DOCKER_REGISTRY=<login>/ DOCKER_PUSH=true make
```

## How to use this image in Makefile

Required variables:

```makefile
PROJECT:=$(subst ${GOPATH}/src/,,$(shell pwd))
```

Use examples:

## protoc

```makefile
.PHONY: proto
proto:
	$(eval $@_source := service/test)
	$(eval $@_target := service/test)

	rm -f ${$@_target}/*.pb.go

	docker run -it --rm \
	-v "$(shell pwd):/go/src/${PROJECT}" \
	-v "${GOPATH}/pkg:/go/pkg" \
	-e "GOPRIVATE=" \
	-e "GOFLAGS=" \
	-w "/go/src/${PROJECT}" \
	dialogs/go-tools-protoc:latest \
	protoc \
	-I=${$@_source} \
	-I=vendor \
	--gogofaster_out=plugins=grpc,\
	Mgoogle/protobuf/empty.proto=github.com/gogo/protobuf/types,\
	:${$@_target} \
	${$@_source}/*.proto
```

## linter

```makefile
.PHONY: linter
linter:
	docker run -it --rm \
	-v "$(shell pwd):/go/src/${PROJECT}" \
	-v "${GOPATH}/pkg:/go/pkg" \
	-e "GOPRIVATE=" \
	-e "GOFLAGS=" \
	-w "/go/src/${PROJECT}" \
	dialogs/go-tools-linter:latest \
	golangci-lint run ./... --exclude "is deprecated"
```

## embedded

```makefile
.PHONY: embedded
embedded:
	$(eval $@_target := ${PROJECT}/db/migrations/test)

	docker run -it --rm \
	-v "$(shell pwd):/go/src/${PROJECT}" \
	-v "${GOPATH}/pkg:/go/pkg" \
	-e "GOPRIVATE=" \
	-e "GOFLAGS=" \
	-w "/go/src/${PROJECT}" \
	dialogs/go-tools-embedded:latest \
	sh -c '\
	rm -fv $($@_target)/static.go && \
	go generate $($@_target)'
```

## mock

```makefile
.PHONY: mock
mock:
	$(eval $@_source := kafka)
	$(eval $@_target := ${$@_source}/mocks)

	docker run -it --rm \
	-v "$(shell pwd):/go/src/${PROJECT}" \
	-v "${GOPATH}/pkg:/go/pkg" \
	-e "GOPRIVATE=" \
	-e "GOFLAGS=" \
	-w "/go/src/${PROJECT}" \
	dialogs/go-tools-mock:latest \
	sh -c '\
	mockery -name=IReader -dir=${$@_source} -recursive=false -output=$($@_target) && \
	mockery -name=IWriter -dir=${$@_source} -recursive=false -output=$($@_target)'
```


## easyjson

```makefile
.PHONY: easyjson
easyjson:
	$(eval $@_target := pkg)

	docker run -it --rm \
	-v "$(shell pwd):/go/src/${PROJECT}" \
	-v "${GOPATH}/pkg:/go/pkg" \
	-e "GOPRIVATE=" \
	-e "GOFLAGS=" \
	-w "/go/src/${PROJECT}" \
	dialogs/go-tools-easyjson:latest \
	sh -c '\
	rm -fv ${$@_target}/*_easyjson.go && \
	easyjson -all ${$@_target}/request.go'
```

## avro

```makefile
.PHONY: avro
avro:
	$(eval $@_target := pkg/schemas)

	docker run -it --rm \
	-v "$(shell pwd):/go/src/${PROJECT}" \
	-v "${GOPATH}/pkg:/go/pkg" \
	-e "GOPRIVATE=" \
	-e "GOFLAGS=" \
	-w "/go/src/${PROJECT}" \
	dialogs/go-tools-avro:latest \
	sh -c '\
	rm -fv ${$@_target}/*.go && \
	gogen-avro --package=schemas ${$@_target} ${$@_target}/*.avsc'
```