Phala deploy dockerfiles
====

This repo contains dockerfiles for deployment

## Usage

### PhalaNode

#### Build

`docker build --build-arg PHALA_GIT_TAG=master -f node.Dockerfile -t phala-node:TAG_NAME .`

#### Run

`docker run -dti --rm --name phala-node -e NODE_NAME=my-phala-node -p 9615:9615 -p 9933:9933 -p 9944:9944 -p 30333:30333 -v $(pwd)/data:/root/data phala-node:TAG_NAME`

### PRuntime

#### Build

> For security reason, unofficial PRuntime build can not be registered to Phala chain network.

Software mode

`docker build --build-arg PHALA_GIT_TAG=master -f pruntime.Dockerfile -t phala-pruntime:TAG_NAME .`

Hardware mode

`docker build --build-arg PHALA_GIT_TAG=master --build-arg SGX_MODE=HW --build-arg IAS_SPID=SPID_FROM_INTEL_PORTAL --build-arg IAS_API_KEY=API_KEY_FROM_INTEL_PORTAL --build-arg IAS_ENV=DEV_OR_PROD --build-arg SGX_SIGN_KEY_URL=URL_TO_FETCH_SIGN_KEY --build-arg SGX_ENCLAVE_CONFIG_URL=URL_TO_FETCH_ENCLAVE_CONFIG -f pruntime.Dockerfile -t phala-pruntime:TAG_NAME .`

#### Run

Software mode

`docker run -dti --rm --name phala-pruntime -p 8000:8000 -v $(pwd)/data:/root/data phala-pruntime:TAG_NAME`

Hardware mode

`docker run -dti --rm --name phala-pruntime -p 8000:8000 -v $(pwd)/data:/root/data --device /dev/sgx/enclave --device /dev/sgx/provision phala-pruntime:TAG_NAME`

### PHost

#### Build

`docker build --build-arg PHALA_GIT_TAG=master -f phost.Dockerfile -t phala-phost:TAG_NAME .`

#### Run

`docker run -dti --rm --name phala-phost -e PRUNTIME_ENDPOINT="http://YOUR_IP:8000" -e PHALA_NODE_WS_ENDPOINT="ws://YOUR_IP:9944" -e MNEMONIC="YOUR_MNEMONIC" -e EXTRA_OPTS="-r" phala-phost:TAG_NAME`

If PhalNode and PRuntime runs in the same PC, you can use `--link` to connect them

`docker run -dti --rm --name phala-phost -e PRUNTIME_ENDPOINT="http://phala-pruntime:8000" -e PHALA_NODE_WS_ENDPOINT="ws://phala-node:9944" -e MNEMONIC="YOUR_MNEMONIC" -e EXTRA_OPTS="-r" --link phala-node --link phala-pruntime phala-phost:TAG_NAME`

Note:

Remember start PHost after PhalaNode and Phost started to accept connections.

If you're using Docker Compose or some sort of, you would adding `-e SLEEP_BEFORE_START=10` to ensure PRuntime started before starting PHost

About `EXTRA_OPTS`:

- `-r` means enable remote attestation flow, this must be set if you wanna join the public chain network, and can only be enabled on Hardware mode

### SGX Detect

`sgx-detect` is a diagnostic tool, it requires you must have SGX hardware mode

#### Build

`docker build -f sgx_detect.Dockerfile -t sgx_detect:TAG_NAME .`

#### Run

`docker run -ti --rm --name sgx_detect --device /dev/sgx/enclave --device /dev/sgx/provision sgx_detect:TAG_NAME`

## Cheatsheets

### Clean build

Add `--no-cache` for a clean build

### Start & stop a container

Start

`docker start phala-node`

Safe stop

`docker stop phala-node`

Force stop

`docker kill phala-node`

### Remove a container

`docker rm phala-node`

### Show logs

`docker logs phala-node`

`docker attach --sig-proxy=false --detach-keys=ctrl-c phala-node`

### Run shell

`docker exec -it phala-node bash`

### Clean up

`docker image prune -a`

## Build, Push, Pull to GitHub

`docker build --build-arg PHALA_GIT_TAG=master -f node.Dockerfile -t docker.pkg.github.com/phala-network/phala-docker/phala-node:TAG_NAME .`

`docker push docker.pkg.github.com/phala-network/phala-docker/phala-node:TAG_NAME`

`docker pull docker.pkg.github.com/phala-network/phala-docker/phala-node:TAG_NAME`

## References

- Proxy and other Systemd relates configurations <https://docs.docker.com/config/daemon/systemd/>
- Manage Docker as a non-root user (avoid `sudo`) <https://docs.docker.com/engine/install/linux-postinstall/>
- Add `--restart=unless-stopped` to `docker run` to improve availability
- You can use `docker create` instead of `docker run` for create the container but not run it immediately

## License

This project is licensed under the terms of the MIT license.
