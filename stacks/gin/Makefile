.PHONY: setup-hooks fmt lint test run

setup-hooks:
	pre-commit install --install-hooks

APP := $(notdir $(abspath .))

fmt:
	gofmt -w .

lint:
	golangci-lint run

test:
	go test ./...

run:
	go run ./cmd/$(APP)
