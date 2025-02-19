GITCOMMIT ?= $(shell git rev-parse HEAD)
GITDATE ?= $(shell git show -s --format='%ct')
VERSION := v0.0.0

LDFLAGSSTRING +=-X main.GitCommit=$(GITCOMMIT)
LDFLAGSSTRING +=-X main.GitDate=$(GITDATE)
LDFLAGSSTRING +=-X github.com/ethereum-optimism/optimism/op-program/version.Version=$(VERSION)
LDFLAGSSTRING +=-X github.com/ethereum-optimism/optimism/op-program/version.Meta=$(VERSION_META)
LDFLAGS := -ldflags "$(LDFLAGSSTRING)"

COMPAT_DIR := temp/compat

op-program: \
	op-program-host \
	op-program-client

op-program-host:
	env GO111MODULE=on GOOS=$(TARGETOS) GOARCH=$(TARGETARCH) go build -v $(LDFLAGS) -o ./bin/op-program ./host/cmd/main.go

op-program-client:
	env GO111MODULE=on GOOS=$(TARGETOS) GOARCH=$(TARGETARCH) go build -v $(LDFLAGS) -o ./bin/op-program-client ./client/cmd/main.go

op-program-client-mips:
	env GO111MODULE=on GOOS=linux GOARCH=mips GOMIPS=softfloat go build -v $(LDFLAGS) -o ./bin/op-program-client.elf ./client/cmd/main.go
	# verify output with: readelf -h bin/op-program-client.elf
	# result is mips32, big endian, R3000

reproducible-prestate:
	@docker build --output ./bin/ --progress plain -f Dockerfile.repro ../
	@echo "Absolute prestate hash:"
	@cat ./bin/prestate-proof.json | jq -r .pre
.PHONY: reproducible-prestate

clean:
	rm -rf bin "$(COMPAT_DIR)"

test:
	go test -v ./...

verify-sepolia: op-program-host op-program-client
	env GO111MODULE=on go run ./verify/sepolia/cmd/sepolia.go --l1 $$SEPOLIA_L1URL --l1.beacon $$SEPOLIA_BEACON_URL --l2 $$SEPOLIA_L2URL --datadir /tmp/test-sepolia

capture-mainnet-genesis: op-program-host op-program-client
	rm -rf "$(COMPAT_DIR)/mainnet-genesis" "$(COMPAT_DIR)/mainnet-genesis.tar.bz"
	env GO111MODULE=on go run ./verify/mainnet/cmd/mainnet.go --l1 $$MAINNET_L1URL --l1.beacon $$MAINNET_BEACON_URL --l2 $$MAINNET_L2URL --datadir "$(COMPAT_DIR)/mainnet-genesis" --l1.head "0x4903424f6cc2cfba7c2bf8c8f48ca46721c963fa64b411cfee3697b781e3e5f1" --l2.start "105235063" --l2.end "105235064"
	tar jcf "$(COMPAT_DIR)/mainnet-genesis.tar.bz" -C "$(COMPAT_DIR)" mainnet-genesis

capture-sepolia-delta: op-program-host op-program-client
	rm -rf "$(COMPAT_DIR)/sepolia-delta" "$(COMPAT_DIR)/sepolia-delta.tar.bz"
	env GO111MODULE=on go run ./verify/sepolia/cmd/sepolia.go --l1 $$SEPOLIA_L1URL --l1.beacon $$SEPOLIA_BEACON_URL --l2 $$SEPOLIA_L2URL --datadir "$(COMPAT_DIR)/sepolia-delta" --l1.head "0x935428728bcfcfeb2e5ba9175fd2890e52831dae221aa4d5dcffed8320edc001" --l2.start "8728200" --l2.end "8728320"
	tar jcf "$(COMPAT_DIR)/sepolia-delta.tar.bz" -C "$(COMPAT_DIR)" sepolia-delta

capture-sepolia-ecotone: op-program-host op-program-client
	rm -rf "$(COMPAT_DIR)/sepolia-ecotone" "$(COMPAT_DIR)/sepolia-ecotone.tar.bz"
	env GO111MODULE=on go run ./verify/sepolia/cmd/sepolia.go --l1 $$SEPOLIA_L1URL --l1.beacon $$SEPOLIA_BEACON_URL --l2 $$SEPOLIA_L2URL --datadir "$(COMPAT_DIR)/sepolia-ecotone" --l1.head "0x5d491a8c1e728a4e70720c09fefdaa083681a9421cd365af85220cf8bd4448a3" --l2.start "9205715" --l2.end "9205815"
	tar jcf "$(COMPAT_DIR)/sepolia-ecotone.tar.bz" -C "$(COMPAT_DIR)" sepolia-ecotone

capture-chain-test-data: capture-mainnet-genesis capture-sepolia-delta capture-sepolia-ecotone

verify-sepolia-delta: op-program-host op-program-client
	./scripts/run-compat.sh "sepolia-delta"

verify-mainnet-genesis: op-program-host op-program-client
	./scripts/run-compat.sh "mainnet-genesis"

verify-sepolia-ecotone: op-program-host op-program-client
	./scripts/run-compat.sh "sepolia-ecotone"

verify-compat: verify-sepolia-delta verify-sepolia-ecotone verify-mainnet-genesis

.PHONY: \
	op-program \
	op-program-host \
	op-program-client \
	op-program-client-mips \
	clean \
	test \
	capture-goerli-verify \
	verify-sepolia \
	capture-mainnet-genesis \
	capture-sepolia-delta \
	capture-sepolia-ecotone \
	capture-chain-test-data \
	run-goerli-verify \
	run-sepolia-verify \
	run-compat
