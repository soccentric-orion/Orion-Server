# Orion-Server Build and Deployment

## 1. Purpose and Scope
Orion-Server is the C++ orchestration/evidence backend. Its root Makefile supports native CMake/Ninja builds, ARM64 client-style Docker builds, and an x86 Ubuntu 22.04 Docker server build. CMake presets, migrations, Docker contexts, and the server runtime scripts provide the surrounding development surfaces.

## 2. Prerequisites
- Linux with GNU Make, CMake, Ninja, C++17 compiler, CTest, and required Drogon/gRPC/PostgreSQL/SQLite dependencies.
- Docker Buildx and QEMU/binfmt for ARM64; Docker for the x86 server image.
- PostgreSQL and the migration configuration for meaningful database tests; the migration test requires `ORION_TEST_DSN`.
- `sshpass`, SSH/SCP access, and reachable targets only for the inherited remote deploy/run targets.

## 3. Setup
```bash
make help
cmake --preset debug
```
The repository also contains `CMakePresets.json`, `.env.example`, `infra`-style/runtime configuration, and migration directories. The build is Docker-based (`make build` → docker-build-server, x86 Ubuntu image, output at `build/docker`).

## 4. Build and Compile
```bash
make configure
make build
make docker-build-server
make docker-build
```
`make build` configures `build/native` with Ninja and builds the server tree. `docker-build-server` builds/uses `orion-dev` on `linux/amd64` and compiles `Orion-Server`; the generic `docker-build` uses the ARM64 `orion` image and shared Orion source mount. Set `JOBS=N` to control native work; Docker caps jobs for emulation.

## 5. Test and Lint
```bash
make test
make docker-test-server
make docker-test
make format
make cppcheck
make doc
```
Native and Docker tests run CTest with failure output. `format`, `cppcheck`, and `doc` invoke the corresponding Ninja targets; install `cmake-format`, `cppcheck`, Doxygen, and Graphviz as required by the target environment. The migration test is conditional on `ORION_TEST_DSN`; without it, the repository's audit notes one skipped migration check rather than a database integration result.

## 6. Packaging and Artifacts
No CPack/package target is defined in this root Makefile. Native outputs are under `build/native`; Docker server output is under `build/docker`; ARM64 shared output is under `build/cross`. Runtime deployment artifacts include the executable, configuration, migrations, and any generated documentation. Container publishing is not implemented here and requires an external registry workflow and credentials.

## 7. Deploy, Install, and Run
```bash
make deploy-a IP_A=192.168.1.3 USER=nvidia PASS='<target-password>'
make deploy-b IP_B=192.168.1.13 USER=nvidia PASS='<target-password>'
make run-a
make run-b
make run APP_NAME arg1
make docker-shell-server
```
The inherited deploy targets build the ARM64 tree, copy `build/cross/lib` and `build/cross/bin` to `/tmp/acu-test`, and run selected/all applications remotely. They do not install the server service or configure PostgreSQL. The x86 Docker shell/build targets are for server development; production installation must provision database DSN, migrations, auth secrets, ports, TLS, and service supervision separately. Never use the placeholder credentials in a real environment.

## 8. Cleanup
```bash
make clean
make docker-clean
make docker-clean-server
```
`clean` removes native and cross build directories. The Docker cleanup targets remove their respective build directories inside containers; `docker-image-server`/`docker-image` may need to remain available for those operations. Preserve database volumes and migration state unless an explicit reset is intended.

## 9. Hardware, Cloud, and Manual Verification Caveats
Server unit tests cover substantial utilities/services but do not prove a complete client-to-server-to-dashboard deployment. Manually verify migrations against PostgreSQL, gRPC streaming/backpressure, REST auth/session and bearer flows, SSE, OTA orchestration, retention, and failure recovery. Hardware is only needed for end-to-end Orion client/HAL interaction; cloud deployment, registry credentials, production secrets, and external object storage are outside this Makefile.
