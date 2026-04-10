This file is a merged representation of the entire codebase, combined into a single document by Repomix.

<file_summary>
This section contains a summary of this file.

<purpose>
This file contains a packed representation of the entire repository's contents.
It is designed to be easily consumable by AI systems for analysis, code review,
or other automated processes.
</purpose>

<file_format>
The content is organized as follows:
1. This summary section
2. Repository information
3. Directory structure
4. Repository files (if enabled)
5. Multiple file entries, each consisting of:
  - File path as an attribute
  - Full contents of the file
</file_format>

<usage_guidelines>
- This file should be treated as read-only. Any changes should be made to the
  original repository files, not this packed version.
- When processing this file, use the file path to distinguish
  between different files in the repository.
- Be aware that this file may contain sensitive information. Handle it with
  the same level of security as you would the original repository.
</usage_guidelines>

<notes>
- Some files may have been excluded based on .gitignore rules and Repomix's configuration
- Binary files are not included in this packed representation. Please refer to the Repository Structure section for a complete list of file paths, including binary files
- Files matching patterns in .gitignore are excluded
- Files matching default ignore patterns are excluded
- Files are sorted by Git change count (files with more changes are at the bottom)
</notes>

</file_summary>

<directory_structure>
.github/
  workflows/
    scripts/
      build.sh
    backport-pr.yml
    build.yml
    codespell.yml
    conventional_commits.yml
    fossa.yml
    stale.yaml
  mergify.yml
  PULL_REQUEST_TEMPLATE.md
app/
  cmd/
    process.go
    start.go
    version.go
package/
  Dockerfile
  exec-logrotate
  instance-manager
  instance-manager-v2-prestop
pkg/
  api/
    backing_image.go
    disk.go
    instance.go
    process.go
  client/
    disk.go
    instance.go
    process_manager.go
    proxy_backing_image.go
    proxy_backup.go
    proxy_metrics.go
    proxy_replica.go
    proxy_snapshot.go
    proxy_types.go
    proxy_volume.go
    proxy.go
    types.go
  disk/
    disk.go
    healthchecker.go
    types.go
  health/
    disk_service_health_probe.go
    health_probe.go
    instance_service_health_probe.go
    proxy_health_probe.go
    spdk_service_health_probe.go
  instance/
    healthchecker.go
    instance.go
    log.go
  meta/
    version.go
  process/
    command.go
    healthchecker.go
    process_manager.go
    process_test.go
    process.go
    version.go
  proxy/
    backing_image.go
    backup.go
    healthchecker.go
    import_backupstores.go
    metrics.go
    proxy.go
    replica.go
    snapshot.go
    volume.go
  types/
    types.go
  util/
    broadcaster/
      broadcaster.go
    grpcutil_test.go
    grpcutil.go
    log.go
    util.go
scripts/
  build
  ci
  entry
  package
  test
  validate
  version
.gitignore
CODE_OF_CONDUCT.md
codecov.yml
Dockerfile.dapper
go.mod
LICENSE
main.go
Makefile
README.md
renovate.json
version
</directory_structure>

<files>
This section contains the contents of the repository's files.

<file path=".github/workflows/scripts/build.sh">
#!/bin/bash

function convert_version_to_major_minor_x() {
    local version="$1"
    if [[ "$version" =~ ^v([0-9]+)\.([0-9]+)\. ]]; then
        echo "v${BASH_REMATCH[1]}.${BASH_REMATCH[2]}.x"
    else
        echo "Invalid version format: $version"
    fi
}

function get_branch() {
    local version_file="version"
    if [[ ! -f $version_file ]]; then
        echo "Error: Version file '$version_file' not found."
        exit 1
    fi

    local version=$(cat "$version_file")
    local branch=$(convert_version_to_major_minor_x "$version")

    # Fetch versions.json from the appropriate branch, fallback to main
    wget -q "https://raw.githubusercontent.com/longhorn/dep-versions/${branch}/versions.json" -O versions.json
    if [ $? -eq 0 ]; then
        echo "${branch}"
    else
        echo "main"
    fi
}
</file>

<file path=".github/workflows/backport-pr.yml">
name: Link-Backport-PR-Issue

on:
  pull_request:
    types: [opened]
    branches:
      - master
      - "v*"

jobs:
  call-workflow:
    uses: longhorn/longhorn/.github/workflows/backport-pr.yml@master
</file>

<file path=".github/workflows/build.yml">
name: build
on:
  push:
    branches:
    - master
    - v*
    tags:
    - v*
  pull_request:
  workflow_dispatch:
jobs:
  build_info:
    name: Collect build info
    runs-on: ubuntu-latest
    outputs:
      version_major: ${{ steps.build_info.outputs.version_major }}
      version_minor: ${{ steps.build_info.outputs.version_minor }}
      version_patch: ${{ steps.build_info.outputs.version_patch }}
      image_tag: ${{ steps.build_info.outputs.image_tag }}

    steps:
    - id: build_info
      name: Declare build info
      run: |
        version_major=''
        version_minor=''
        version_patch=''
        image_tag=''

        branch=${GITHUB_HEAD_REF:-${GITHUB_REF#refs/heads/}}
        ref=${{ github.ref }}
        if [[ "$ref" =~ 'refs/tags/' ]]; then
          version=$(sed -E 's/^v([0-9]*\.[0-9]*\.[0-9]*).*$/\1/' <<<${{ github.ref_name }} )
          version_major=$(cut -d. -f1 <<<$version)
          version_minor=$(cut -d. -f2 <<<$version)
          version_patch=$(cut -d. -f3 <<<$version)
          image_tag=${{ github.ref_name }}
        elif [[ "$ref" =~ 'refs/heads/' ]]; then
          image_tag="${branch}-head"
        fi

        echo "version_major=${version_major}" >>$GITHUB_OUTPUT
        echo "version_minor=${version_minor}" >>$GITHUB_OUTPUT
        echo "version_patch=${version_patch}" >>$GITHUB_OUTPUT
        echo "image_tag=${image_tag}" >>$GITHUB_OUTPUT

        cat <<EOF
        version_major=${version_major}
        version_minor=${version_minor}
        version_patch=${version_patch}
        image_tag=${image_tag}
        EOF

  build-amd64-binaries:
    name: Build AMD64 binaries
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    # Build binaries
    - name: Run make ci
      run: SKIP_TASKS=package make ci

    - uses: codecov/codecov-action@v4
      with:
        files: ./coverage.out
        flags: unittests
        token: ${{ secrets.CODECOV_TOKEN }}

    - name: Upload binaries
      uses: actions/upload-artifact@v4
      with:
        name: binaries_amd64_artifact
        path: ./bin/*

  build-arm64-binaries:
    name: Build ARM64 binaries
    runs-on: longhorn-infra-oracle-arm64-runners
    steps:
    - name: Install make curl git
      run: |
        sudo apt update
        sudo apt-get -y install make curl git

    - name: Checkout code
      uses: actions/checkout@v4

    # Build binaries
    - name: Run make ci
      run: sudo SKIP_TASKS=package make ci

    - name: Upload binaries
      uses: actions/upload-artifact@v4
      with:
        name: binaries_arm64_artifact
        path: ./bin/*

  build-push-amd64-images:
    name: Build and push AMD64 images
    runs-on: ubuntu-latest
    if: ${{ startsWith(github.ref, 'refs/heads/') || startsWith(github.ref, 'refs/tags/') }}
    needs: [build_info, build-amd64-binaries]
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up QEMU
      uses: docker/setup-qemu-action@v3
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Download binaries
      uses: actions/download-artifact@v4
      with:
        name: binaries_amd64_artifact
        path: ./bin/

    - name: Add executable permission
      run: |
        chmod +x ./bin/*

    - name: Login to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    # longhornio/longhorn-instance-manager image
    - name: Build and publish image
      env:
        REPO: docker.io/longhornio
        TAG: ${{ needs.build_info.outputs.image_tag }}-amd64
        TARGET_PLATFORMS: linux/amd64
      run: make workflow-image-build-push

  build-push-arm64-images:
    name: Build and push ARM64 images
    runs-on: longhorn-infra-oracle-arm64-runners
    if: ${{ startsWith(github.ref, 'refs/heads/') || startsWith(github.ref, 'refs/tags/') }}
    needs: [build_info, build-arm64-binaries]
    steps:
    - name: Install make curl git
      run: |
        sudo apt update
        sudo apt-get -y install make curl git

    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up QEMU
      uses: docker/setup-qemu-action@v3
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Download binaries
      uses: actions/download-artifact@v4
      with:
        name: binaries_arm64_artifact
        path: ./bin/

    - name: Add executable permission
      run: |
        chmod +x ./bin/*

    - name: Login to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    # longhornio/longhorn-instance-manager image
    - name: Build and publish image
      env:
        REPO: docker.io/longhornio
        TAG: ${{ needs.build_info.outputs.image_tag }}-arm64
        TARGET_PLATFORMS: linux/arm64
      run: make workflow-image-build-push

  manifest-image:
    name: Manifest images
    runs-on: ubuntu-latest
    needs: [build_info, build-push-amd64-images, build-push-arm64-images]
    if: ${{ startsWith(github.ref, 'refs/heads/') || startsWith(github.ref, 'refs/tags/') }}
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Login to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    # longhornio/longhorn-instance-manager image
    - name: docker-pull-manifest-longhorn-instance-manager
      env:
        REPO: docker.io/longhornio
        TAG: ${{ needs.build_info.outputs.image_tag }}
      run: make workflow-manifest-image
</file>

<file path=".github/workflows/codespell.yml">
name: Codespell

on:
  pull_request:
    branches:
    - master
    - "v*.*.*"

jobs:
  codespell:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v4
      with:
        fetch-depth: 1
    - name: Check code spell
      uses: codespell-project/actions-codespell@v2
      with:
        check_filenames: true
        skip: "./proto,*/**.yaml,*/**.yml,./scripts,./vendor,MAINTAINERS,LICENSE,go.mod,go.sum"
</file>

<file path=".github/workflows/conventional_commits.yml">
name: Conventional Commits

on:
  pull_request_target:
    types:
    - opened
    - edited
    - synchronize
    - reopened

permissions:
  pull-requests: read

jobs:
  commit-lint:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
    - name: Lint Commits
      uses: wagoid/commitlint-github-action@v6
    - name: Lint Pull Request
      uses: amannn/action-semantic-pull-request@v5.5.3
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        types: |
          feat
          fix
          docs
          style
          refactor
          perf
          test
          chore
          vendor
          build
          ci
          revert
          BREAKING
</file>

<file path=".github/workflows/fossa.yml">
name: fossa
on:
  push:
    branches:
      - master
      - v*
    tags:
      - v*
  pull_request:
    branches:
      - master
      - v*
  workflow_dispatch: {}

permissions: {}

jobs:
  fossa-scan:
    if: github.repository == 'longhorn/longhorn-instance-manager' # FOSSA is not intended to run on forks.
    runs-on: ubuntu-latest
    permissions:
      contents: read
    env:
      FOSSA_API_KEY: ${{ secrets.FOSSA_API_KEY }}
    steps:
      - name: "Checkout code"
        uses: actions/checkout@v4

      - name: "Run FOSSA Scan"
        uses: fossas/fossa-action@v1.8.0 # Use a specific version if locking is preferred
        with:
          api-key: ${{ secrets.FOSSA_API_KEY }}
          project: longhorn-instance-manager
</file>

<file path=".github/workflows/stale.yaml">
name: 'Close stale issues and PRs'

on:
  workflow_dispatch:
  schedule:
    - cron: '30 1 * * *'

jobs:
  call-workflow:
    uses: longhorn/longhorn/.github/workflows/stale.yaml@master
</file>

<file path=".github/mergify.yml">
pull_request_rules:
- name: Automatically merge PRs
  conditions:
  - check-success="Build AMD64 binaries"
  - check-success="Build ARM64 binaries"
  - "#approved-reviews-by>=2"
  - approved-reviews-by=@longhorn/maintainer
  actions:
    merge:
      method: rebase
      
- name: Automatically merge Renovate PRs
  conditions:
  - check-success="Build AMD64 binaries"
  - check-success="Build ARM64 binaries"
  - author = renovate[bot]
  actions:
    merge:
      method: rebase

- name: Automatically approve Renovate PRs
  conditions:
  - check-success="Build AMD64 binaries"
  - check-success="Build ARM64 binaries"
  - author = renovate[bot]
  actions:
    review:
      type: APPROVE

- name: Ask to resolve conflict
  conditions:
  - conflict
  actions:
    comment:
      message: This pull request is now in conflict. Could you fix it @{{author}}? 🙏
</file>

<file path=".github/PULL_REQUEST_TEMPLATE.md">
#### Which issue(s) this PR fixes:
<!--
Use `Issue #<issue number>` or `Issue longhorn/longhorn#<issue number>` or `Issue (paste link of issue)`. DON'T use `Fixes #<issue number>` or `Fixes (paste link of issue)`, as it will automatically close the linked issue when the PR is merged.
-->
Issue #

#### What this PR does / why we need it:

#### Special notes for your reviewer:

#### Additional documentation or context
</file>

<file path="app/cmd/process.go">
package cmd

import (
	"context"
	"path/filepath"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"github.com/urfave/cli"

	"github.com/longhorn/longhorn-instance-manager/pkg/client"
	"github.com/longhorn/longhorn-instance-manager/pkg/util"
)

func ProcessCmd() cli.Command {
	return cli.Command{
		Name: "process",
		Subcommands: []cli.Command{
			ProcessCreateCmd(),
			ProcessDeleteCmd(),
			ProcessGetCmd(),
			ProcessListCmd(),
			ProcessReplaceCmd(),
		},
	}
}

func ProcessCreateCmd() cli.Command {
	return cli.Command{
		Name: "create",
		Flags: []cli.Flag{
			cli.StringFlag{
				Name: "name",
			},
			cli.StringFlag{
				Name: "binary",
			},
			cli.IntFlag{
				Name: "port-count",
			},
			cli.StringSliceFlag{
				Name:  "port-args",
				Usage: "Automatically add additional arguments when starting the process. In case of space, use `,` instead.",
			},
		},
		Action: func(c *cli.Context) {
			if err := createProcess(c); err != nil {
				logrus.WithError(err).Fatal("Error running process create command")
			}
		},
	}
}

func createProcess(c *cli.Context) error {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	cli, err := getProcessManagerClient(c, ctx, cancel)
	if err != nil {
		return errors.Wrap(err, "failed to initialize ProcessManager client")
	}
	defer func() {
		if closeErr := cli.Close(); closeErr != nil {
			logrus.WithError(closeErr).Warn("Failed to close ProcessManager client")
		}
	}()

	process, err := cli.ProcessCreate(c.String("name"), c.String("binary"),
		c.Int("port-count"), c.Args(), c.StringSlice("port-args"))
	if err != nil {
		return errors.Wrap(err, "failed to create process")
	}
	return util.PrintJSON(process)
}

func ProcessDeleteCmd() cli.Command {
	return cli.Command{
		Name: "delete",
		Flags: []cli.Flag{
			cli.StringFlag{
				Name: "name",
			},
			cli.StringFlag{
				Name:     "uuid",
				Required: false,
				Usage:    "Validate the process UUID. If provided, the process will be deleted only when both name and UUID are matched.",
			},
		},
		Action: func(c *cli.Context) {
			if err := deleteProcess(c); err != nil {
				logrus.WithError(err).Fatal("Error running process delete command")
			}
		},
	}
}

func deleteProcess(c *cli.Context) error {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	cli, err := getProcessManagerClient(c, ctx, cancel)
	if err != nil {
		return errors.Wrap(err, "failed to initialize ProcessManager client")
	}
	defer func() {
		if closeErr := cli.Close(); closeErr != nil {
			logrus.WithError(closeErr).Warn("Failed to close ProcessManager client")
		}
	}()

	process, err := cli.ProcessDelete(c.String("name"), c.String("uuid"))
	if err != nil {
		return errors.Wrap(err, "failed to delete process")
	}
	return util.PrintJSON(process)
}

func ProcessGetCmd() cli.Command {
	return cli.Command{
		Name: "get",
		Flags: []cli.Flag{
			cli.StringFlag{
				Name: "name",
			},
		},
		Action: func(c *cli.Context) {
			if err := getProcess(c); err != nil {
				logrus.WithError(err).Fatal("Error running process get command")
			}
		},
	}
}

func getProcess(c *cli.Context) error {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	cli, err := getProcessManagerClient(c, ctx, cancel)
	if err != nil {
		return errors.Wrap(err, "failed to initialize ProcessManager client")
	}
	defer func() {
		if closeErr := cli.Close(); closeErr != nil {
			logrus.WithError(closeErr).Warn("Failed to close ProcessManager client")
		}
	}()

	process, err := cli.ProcessGet(c.String("name"))
	if err != nil {
		return errors.Wrap(err, "failed to delete process")
	}
	return util.PrintJSON(process)
}

func ProcessListCmd() cli.Command {
	return cli.Command{
		Name:      "list",
		ShortName: "ls",
		Action: func(c *cli.Context) {
			if err := listProcess(c); err != nil {
				logrus.WithError(err).Fatal("Error running engine stop command")
			}
		},
	}
}

func listProcess(c *cli.Context) error {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	cli, err := getProcessManagerClient(c, ctx, cancel)
	if err != nil {
		return errors.Wrap(err, "failed to initialize ProcessManager client")
	}
	defer func() {
		if closeErr := cli.Close(); closeErr != nil {
			logrus.WithError(closeErr).Warn("Failed to close ProcessManager client")
		}
	}()

	processes, err := cli.ProcessList()
	if err != nil {
		return errors.Wrap(err, "failed to list processes")
	}
	return util.PrintJSON(processes)
}

func ProcessReplaceCmd() cli.Command {
	return cli.Command{
		Name: "replace",
		Flags: []cli.Flag{
			cli.StringFlag{
				Name: "name",
			},
			cli.StringFlag{
				Name: "binary",
			},
			cli.IntFlag{
				Name: "port-count",
			},
			cli.StringSliceFlag{
				Name:  "port-args",
				Usage: "Automatically add additional arguments when starting the process. In case of space, use `,` instead.",
			},
			cli.StringFlag{
				Name:  "terminate-signal",
				Usage: "The signal used to terminate the old process",
				Value: "SIGHUP",
			},
		},
		Action: func(c *cli.Context) {
			if err := replaceProcess(c); err != nil {
				logrus.WithError(err).Fatal("Error running engine replace command")
			}
		},
	}
}

func replaceProcess(c *cli.Context) error {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	cli, err := getProcessManagerClient(c, ctx, cancel)
	if err != nil {
		return errors.Wrap(err, "failed to initialize ProcessManager client")
	}
	defer func() {
		if closeErr := cli.Close(); closeErr != nil {
			logrus.WithError(closeErr).Warn("Failed to close ProcessManager client")
		}
	}()

	process, err := cli.ProcessReplace(c.String("name"), c.String("binary"),
		c.Int("port-count"), c.Args(), c.StringSlice("port-args"), c.String("terminate-signal"))
	if err != nil {
		return errors.Wrap(err, "failed to replace processes")
	}
	return util.PrintJSON(process)
}

func getProcessManagerClient(c *cli.Context, ctx context.Context, ctxCancel context.CancelFunc) (*client.ProcessManagerClient, error) {
	url := c.GlobalString("url")
	tlsDir := c.GlobalString("tls-dir")

	if tlsDir != "" {
		imClient, err := client.NewProcessManagerClientWithTLS(ctx, ctxCancel, url,
			filepath.Join(tlsDir, "ca.crt"),
			filepath.Join(tlsDir, "tls.crt"),
			filepath.Join(tlsDir, "tls.key"),
			"longhorn-backend.longhorn-system")
		if err == nil {
			return imClient, err
		}
		logrus.WithError(err).Info("Falling back to non tls ProcessManager client")
	}

	return client.NewProcessManagerClient(ctx, ctxCancel, url, nil)
}
</file>

<file path="app/cmd/start.go">
package cmd

import (
	"context"
	"crypto/tls"
	"net"
	"net/http"
	_ "net/http/pprof" // for runtime profiling
	"os"
	"os/signal"
	"path/filepath"
	"strconv"
	"strings"
	"syscall"
	"time"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"github.com/urfave/cli"
	"golang.org/x/sync/errgroup"
	"google.golang.org/grpc"
	"google.golang.org/grpc/keepalive"
	"google.golang.org/grpc/reflection"

	healthpb "google.golang.org/grpc/health/grpc_health_v1"

	"k8s.io/mount-utils"

	engineutil "github.com/longhorn/longhorn-engine/pkg/util"
	spdk "github.com/longhorn/longhorn-spdk-engine/pkg/spdk"
	spdkutil "github.com/longhorn/longhorn-spdk-engine/pkg/util"
	rpc "github.com/longhorn/types/pkg/generated/imrpc"
	spdkrpc "github.com/longhorn/types/pkg/generated/spdkrpc"

	"github.com/longhorn/longhorn-instance-manager/pkg/disk"
	"github.com/longhorn/longhorn-instance-manager/pkg/health"
	"github.com/longhorn/longhorn-instance-manager/pkg/instance"
	"github.com/longhorn/longhorn-instance-manager/pkg/process"
	"github.com/longhorn/longhorn-instance-manager/pkg/proxy"
	"github.com/longhorn/longhorn-instance-manager/pkg/types"
	"github.com/longhorn/longhorn-instance-manager/pkg/util"
)

const (
	spdkTgtStopTimeout = 120 * time.Second
)

func StartCmd() cli.Command {
	return cli.Command{
		Name: "daemon",
		Flags: []cli.Flag{
			cli.StringFlag{
				Name:  "listen",
				Value: "localhost:8500",
				Usage: "specifies the server endpoint to listen on supported protocols are 'tcp' and 'unix'. The proxy server will be listening on the next port.",
			},
			cli.StringFlag{
				Name:  "logs-dir",
				Value: "/var/log/instances",
			},
			cli.StringFlag{
				Name:  "port-range",
				Value: "10000-20000",
			},
			cli.StringFlag{
				Name:  "spdk-port-range",
				Value: "20001-30000",
			},
			cli.BoolFlag{
				Name:  "spdk-enabled",
				Usage: "enable SPDK support",
			},
		},
		Action: func(c *cli.Context) {
			if err := start(c); err != nil {
				logrus.WithError(err).Fatal("Failed to run start command")
			}
		},
	}
}

func cleanup(pm *process.Manager) {
	logrus.Infof("Trying to gracefully shut down %v", types.ProcessManagerGrpcService)

	pmResp, err := pm.ProcessList(context.TODO(), &rpc.ProcessListRequest{})
	if err != nil {
		logrus.WithError(err).Errorf("Failed to list processes before shutting down %v", types.ProcessManagerGrpcService)
		return
	}
	for _, p := range pmResp.Processes {
		if _, err := pm.ProcessDelete(context.TODO(), &rpc.ProcessDeleteRequest{
			Name: p.Spec.Name,
		}); err != nil {
			logrus.WithError(err).Errorf("Failed to delete process %s", p.Spec.Name)
		}
	}

	for i := 0; i < types.WaitCount; i++ {
		pmResp, err := pm.ProcessList(context.TODO(), &rpc.ProcessListRequest{})
		if err != nil {
			logrus.WithError(err).Errorf("Failed to list processes when shutting down %v", types.ProcessManagerGrpcService)
			break
		}
		if len(pmResp.Processes) == 0 {
			logrus.Info("Shut down all processes successfully")
			return
		}
		time.Sleep(types.WaitInterval)
	}

	logrus.Errorf("Failed to clean up all processes for %s graceful shutdown", types.ProcessManagerGrpcService)
}

func unfreezeFilesystems() error {
	// We do not need to switch to the host mount namespace to get mount points here. Usually, longhorn-engine runs in a
	// container that has / bind mounted to /host with at least HostToContainer (rslave) propagation.
	// - If it does not, we likely can't do a namespace swap anyway, since we don't have access to /host/proc.
	// - If it does, we just need to know where in the container we can access the mount points to unfreeze the file
	//   system.
	mounter := mount.New("")
	mountPoints, err := mounter.List()
	if err != nil {
		return errors.Wrap(err, "failed to list mount points while starting up")
	}

	for _, mountPoint := range mountPoints {
		if strings.Contains(mountPoint.Device, engineutil.DevicePathPrefix) {
			// We do not actually expect any filesystems to be frozen. This is a best effort attempt to unfreeze them
			// if somehow instance manager crashed at the wrong moment during a snapshot.
			unfroze, err := engineutil.UnfreezeFilesystem(mountPoint.Path, nil)
			if err != nil {
				logrus.WithError(err).Warnf("Failed to unfreeze filesystem mounted at %v", mountPoint)
			}
			if unfroze {
				logrus.Warnf("Unfroze filesystem mounted at %v", mountPoint)
			}
		}
	}
	return nil
}

func start(c *cli.Context) (err error) {
	listen := c.String("listen")
	logsDir := c.String("logs-dir")
	processPortRange := c.String("port-range")
	spdkPortRange := c.String("spdk-port-range")
	spdkEnabled := c.Bool("spdk-enabled")

	defer func() {
		if spdkEnabled {
			logrus.Infof("Stopping spdk_tgt daemon")
			if err := spdkutil.StopSPDKTgtDaemon(spdkTgtStopTimeout); err != nil {
				logrus.WithError(err).Error("Failed to stop spdk_tgt daemon")
			}
		}
	}()

	if err := util.SetUpLogger(logsDir); err != nil {
		return err
	}

	if !spdkEnabled {
		if err := unfreezeFilesystems(); err != nil {
			return err
		}
	}

	// setup tls config
	var tlsConfig *tls.Config
	tlsDir := c.GlobalString("tls-dir")
	if tlsDir != "" {
		tlsConfig, err = util.LoadServerTLS(
			filepath.Join(tlsDir, "ca.crt"),
			filepath.Join(tlsDir, "tls.crt"),
			filepath.Join(tlsDir, "tls.key"),
			"longhorn-backend.longhorn-system")
		if err != nil {
			logrus.WithError(err).Warnf("Failed to add TLS key pair from %v", tlsDir)
		}
	}

	if tlsConfig != nil {
		logrus.Info("Creating gRPC server with mtls auth")
	} else {
		logrus.Info("Creating gRPC server with no auth")
	}

	go func() {
		debugAddress := ":6060"
		debugHandler := http.DefaultServeMux
		logrus.Infof("Debug pprof server listening on %s", debugAddress)
		if err := http.ListenAndServe(debugAddress, debugHandler); err != nil && err != http.ErrServerClosed {
			logrus.Errorf("ListenAndServe: %s", err)
		}
	}()

	addresses, err := getServiceAddresses(listen)
	if err != nil {
		logrus.WithError(err).Error("Failed to get service addresses")
		return err
	}

	// Create gRPC servers
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	servers := map[string]*grpc.Server{}
	listeners := map[string]net.Listener{}

	// Start disk server
	diskGRPCServer, diskGRPCListener, err := setupDiskGRPCServer(ctx, addresses[types.DiskGrpcService], addresses[types.SpdkGrpcService], spdkEnabled)
	if err != nil {
		logrus.WithError(err).Errorf("Failed to setup %s", types.DiskGrpcService)
		return err
	}
	servers[types.DiskGrpcService] = diskGRPCServer
	listeners[types.DiskGrpcService] = diskGRPCListener

	// Start instance server
	instanceGRPCServer, instanceRPCListener, err := setupInstanceGRPCServer(ctx, logsDir,
		addresses[types.InstanceGrpcService], addresses[types.ProcessManagerGrpcService],
		addresses[types.SpdkGrpcService], tlsConfig, spdkEnabled)
	if err != nil {
		logrus.WithError(err).Errorf("Failed to set up %s", types.InstanceGrpcService)
		return err
	}
	servers[types.InstanceGrpcService] = instanceGRPCServer
	listeners[types.InstanceGrpcService] = instanceRPCListener

	// Start proxy server
	proxyGRPCServer, proxyGRPCListener, err := setupProxyGRPCServer(ctx, logsDir,
		addresses[types.ProxyGRPCService], addresses[types.DiskGrpcService], addresses[types.SpdkGrpcService], tlsConfig)
	if err != nil {
		logrus.WithError(err).Errorf("Failed to set up %s", types.ProxyGRPCService)
		return err
	}
	servers[types.ProxyGRPCService] = proxyGRPCServer
	listeners[types.ProxyGRPCService] = proxyGRPCListener

	// Start process-manager server
	pm, pmGRPCServer, pmGRPCListener, err := setupProcessManagerGRPCServer(ctx, processPortRange, logsDir, addresses[types.ProcessManagerGrpcService])
	if err != nil {
		logrus.WithError(err).Errorf("Failed to set up %s", types.ProcessManagerGrpcService)
		return err
	}
	servers[types.ProcessManagerGrpcService] = pmGRPCServer
	listeners[types.ProcessManagerGrpcService] = pmGRPCListener

	// Start spdk server
	if spdkEnabled {
		spdkGRPCServer, spdkGRPCListener, err := setupSPDKGRPCServer(ctx, spdkPortRange, addresses[types.SpdkGrpcService])
		if err != nil {
			logrus.WithError(err).Errorf("Failed to set up %s", types.SpdkGrpcService)
			return err
		}
		servers[types.SpdkGrpcService] = spdkGRPCServer
		listeners[types.SpdkGrpcService] = spdkGRPCListener
	}

	g, _ := errgroup.WithContext(ctx)

	// Register signal handler
	sigs := make(chan os.Signal, 1)
	signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)
	g.Go(func() error {
		sig := <-sigs
		logrus.Infof("Instance Manager received %v to exit", sig)

		for _, server := range servers {
			server.Stop()
		}
		return nil
	})

	// Start gRPC servers
	for name, server := range servers {
		name, server := name, server
		g.Go(func() error {
			defer func() {
				// Send SIGTERM to stop other grpc servers
				select {
				case sigs <- syscall.SIGTERM:
					logrus.Infof("Instance Manager sent %v to exit", syscall.SIGTERM)
				default:
					logrus.Infof("Instance Manager already sent %v to exit", syscall.SIGTERM)
				}
			}()

			listener := listeners[name]
			address := addresses[name]

			logrus.Infof("%s listening to %v", name, address)
			err := server.Serve(listener)
			if err != nil {
				logrus.WithError(err).Errorf("%s failed to serve", name)
			}

			if name == types.ProcessManagerGrpcService {
				cleanup(pm)
			}

			logrus.Infof("Stopped %s", name)
			return err
		})
	}

	if err := g.Wait(); err != nil {
		logrus.WithError(err).Error("Instance Manager exited with error")
	}

	return nil
}

func getServiceAddresses(listen string) (addresses map[string]string, err error) {
	host, port, err := net.SplitHostPort(listen)
	if err != nil {
		return nil, err
	}

	intPort, err := strconv.Atoi(port)
	if err != nil {
		return nil, err
	}

	return map[string]string{
		types.ProcessManagerGrpcService: net.JoinHostPort(host, strconv.Itoa(intPort)),
		types.ProxyGRPCService:          net.JoinHostPort(host, strconv.Itoa(intPort+1)),
		types.DiskGrpcService:           net.JoinHostPort(host, strconv.Itoa(intPort+2)),
		types.InstanceGrpcService:       net.JoinHostPort(host, strconv.Itoa(intPort+3)),
		types.SpdkGrpcService:           net.JoinHostPort(host, strconv.Itoa(intPort+4)),
	}, nil
}

func setupDiskGRPCServer(ctx context.Context, listen, spdkServiceAddress string, spdkEnabled bool) (*grpc.Server, net.Listener, error) {
	srv, err := disk.NewServer(ctx, spdkEnabled, spdkServiceAddress)
	if err != nil {
		return nil, nil, err
	}
	hc := health.NewDiskHealthCheckServer(srv)

	grpcServer, rpcListener, err := util.NewServer(listen, nil,
		grpc.KeepaliveEnforcementPolicy(keepalive.EnforcementPolicy{
			MinTime:             10 * time.Second,
			PermitWithoutStream: true,
		}),
	)
	if err != nil {
		return nil, nil, errors.Wrapf(err, "failed to setup %s", types.DiskGrpcService)
	}

	rpc.RegisterDiskServiceServer(grpcServer, srv)
	healthpb.RegisterHealthServer(grpcServer, hc)
	reflection.Register(grpcServer)

	return grpcServer, rpcListener, nil
}

func setupSPDKGRPCServer(ctx context.Context, portRange, listen string) (*grpc.Server, net.Listener, error) {
	portStart, portEnd, err := util.ParsePortRange(portRange)
	if err != nil {
		return nil, nil, err
	}

	srv, err := spdk.NewServer(ctx, portStart, portEnd)
	if err != nil {
		return nil, nil, err
	}
	hc := health.NewSPDKHealthCheckServer(srv)

	grpcServer, grpcListener, err := util.NewServer(listen, nil,
		grpc.KeepaliveEnforcementPolicy(keepalive.EnforcementPolicy{
			MinTime:             10 * time.Second,
			PermitWithoutStream: true,
		}),
	)
	if err != nil {
		return nil, nil, errors.Wrapf(err, "failed to setup %s", types.SpdkGrpcService)
	}

	spdkrpc.RegisterSPDKServiceServer(grpcServer, srv)
	healthpb.RegisterHealthServer(grpcServer, hc)
	reflection.Register(grpcServer)

	return grpcServer, grpcListener, nil
}

func setupProxyGRPCServer(ctx context.Context, logsDir, listen, diskServiceAddress, spdkServiceAddress string, tlsConfig *tls.Config) (*grpc.Server, net.Listener, error) {
	// TODO: skip proxy for replica instance manager pod
	srv, err := proxy.NewProxy(ctx, logsDir, diskServiceAddress, spdkServiceAddress)
	if err != nil {
		return nil, nil, err
	}
	hc := health.NewProxyHealthCheckServer(srv)

	grpcProxyServer, grpcProxyListener, err := util.NewServer(listen, tlsConfig,
		grpc.KeepaliveEnforcementPolicy(keepalive.EnforcementPolicy{
			MinTime:             10 * time.Second,
			PermitWithoutStream: true,
		}),
	)
	if err != nil {
		return nil, nil, errors.Wrapf(err, "failed to setup %s", types.ProxyGRPCService)
	}

	rpc.RegisterProxyEngineServiceServer(grpcProxyServer, srv)
	healthpb.RegisterHealthServer(grpcProxyServer, hc)
	reflection.Register(grpcProxyServer)

	return grpcProxyServer, grpcProxyListener, nil
}

func setupProcessManagerGRPCServer(ctx context.Context, portRange, logsDir, listen string) (*process.Manager, *grpc.Server, net.Listener, error) {
	srv, err := process.NewManager(ctx, portRange, logsDir)
	if err != nil {
		return nil, nil, nil, err
	}
	hc := health.NewHealthCheckServer(srv)

	grpcServer, grpcListener, err := util.NewServer(listen, nil,
		grpc.KeepaliveEnforcementPolicy(keepalive.EnforcementPolicy{
			MinTime:             10 * time.Second,
			PermitWithoutStream: true,
		}),
	)
	if err != nil {
		return nil, nil, nil, errors.Wrapf(err, "failed to setup %s", types.ProcessManagerGrpcService)
	}

	rpc.RegisterProcessManagerServiceServer(grpcServer, srv)
	healthpb.RegisterHealthServer(grpcServer, hc)
	reflection.Register(grpcServer)

	return srv, grpcServer, grpcListener, nil
}

func setupInstanceGRPCServer(ctx context.Context, logsDir, listen, processManagerServiceAddress, spdkServiceAddress string, tlsConfig *tls.Config, spdkEnabled bool) (*grpc.Server, net.Listener, error) {
	srv, err := instance.NewServer(ctx, logsDir, processManagerServiceAddress, spdkServiceAddress, spdkEnabled)
	if err != nil {
		return nil, nil, err
	}
	hc := health.NewInstanceHealthCheckServer(srv)

	grpcServer, grpcListener, err := util.NewServer(listen, tlsConfig,
		grpc.KeepaliveEnforcementPolicy(keepalive.EnforcementPolicy{
			MinTime:             10 * time.Second,
			PermitWithoutStream: true,
		}),
	)
	if err != nil {
		return nil, nil, errors.Wrapf(err, "failed to setup %s", types.InstanceGrpcService)
	}

	rpc.RegisterInstanceServiceServer(grpcServer, srv)
	healthpb.RegisterHealthServer(grpcServer, hc)
	reflection.Register(grpcServer)

	return grpcServer, grpcListener, nil
}
</file>

<file path="app/cmd/version.go">
package cmd

import (
	"context"
	"encoding/json"
	"fmt"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"github.com/urfave/cli"

	"github.com/longhorn/longhorn-instance-manager/pkg/meta"
)

func VersionCmd() cli.Command {
	return cli.Command{
		Name: "version",
		Flags: []cli.Flag{
			cli.BoolFlag{
				Name: "client-only",
			},
		},
		Action: func(c *cli.Context) {
			if err := version(c); err != nil {
				logrus.WithError(err).Fatal("Error running info command")
			}
		},
	}
}

type VersionOutput struct {
	ClientVersion *meta.VersionOutput `json:"clientVersion"`
	ServerVersion *meta.VersionOutput `json:"serverVersion"`
}

func version(c *cli.Context) error {
	clientVersion := meta.GetVersion()
	v := VersionOutput{ClientVersion: &clientVersion}

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	if !c.Bool("client-only") {
		cli, err := getProcessManagerClient(c, ctx, cancel)
		if err != nil {
			return errors.Wrap(err, "failed to initialize ProcessManagerClient")
		}
		defer func() {
			if closeErr := cli.Close(); closeErr != nil {
				logrus.WithError(closeErr).Warn("Failed to close ProcessManagerClient")
			}
		}()

		version, err := cli.VersionGet()
		if err != nil {
			return err
		}
		v.ServerVersion = version
	}
	output, err := json.MarshalIndent(v, "", "\t")
	if err != nil {
		return err
	}

	fmt.Println(string(output))
	return nil
}
</file>

<file path="package/Dockerfile">
# Stage 1: build binary from go source code
FROM  registry.suse.com/bci/golang:1.25 AS gobuilder

ARG ARCH=amd64
ARG SRC_BRANCH=master
ARG SRC_TAG

RUN zypper -n ref && \
    zypper update -y

RUN zypper -n install jq wget

ENV GOLANG_ARCH_amd64=amd64 GOLANG_ARCH_arm64=arm64 GOLANG_ARCH_s390x=s390x GOLANG_ARCH=GOLANG_ARCH_${ARCH} \
    GOPATH=/go PATH=/go/bin:/usr/local/go/bin:${PATH} SHELL=/bin/bash
RUN go install golang.org/x/lint/golint@latest

# If TAG is explicitly set and exists in the repo, switch to the tag
RUN git clone https://github.com/longhorn/dep-versions.git -b ${SRC_BRANCH} /usr/src/dep-versions && \
    cd /usr/src/dep-versions && \
    if [ -n "${SRC_TAG}" ] && git show-ref --tags ${SRC_TAG} > /dev/null 2>&1; then \
        echo "Checking out tag ${SRC_TAG}"; \
        cd /usr/src/dep-versions && git checkout tags/${SRC_TAG}; \
    fi

# Build go-spdk-helper
RUN export REPO_OVERRIDE="" && \
    export COMMIT_ID_OVERRIDE="" && \
    bash /usr/src/dep-versions/scripts/build-go-spdk-helper.sh "${REPO_OVERRIDE}" "${COMMIT_ID_OVERRIDE}"

# Install grpc_health_probe
RUN export GRPC_HEALTH_PROBE_DOWNLOAD_URL=$(wget -qO- https://api.github.com/repos/grpc-ecosystem/grpc-health-probe/releases/latest | jq -r '.assets[] | select(.name | test("linux.*'"${ARCH}"'"; "i")) | .browser_download_url') && \
    wget ${GRPC_HEALTH_PROBE_DOWNLOAD_URL} -O /usr/local/bin/grpc_health_probe && \
    chmod +x /usr/local/bin/grpc_health_probe

# Stage 2: build binary from c source code
FROM registry.suse.com/bci/bci-base:15.7 AS cbuilder

ARG ARCH=amd64
ARG SRC_BRANCH=master
ARG SRC_TAG

# Install build dependencies from zypper
RUN zypper -n ref && \
    zypper update -y

RUN zypper -n install cmake gcc gcc13 xsltproc docbook-xsl-stylesheets git python311 python311-pip fuse3-devel jq nasm

RUN ln -sf /usr/bin/python3.11 /usr/local/bin/python && \
    ln -sf /usr/bin/python3.11 /usr/local/bin/python3 && \
    ln -sf /usr/bin/pip3.11 /usr/local/bin/pip3 && \
    ln -sf /usr/bin/pip3.11 /usr/local/bin/pip

# Install dependencies defined in dep-versions
RUN git clone https://github.com/longhorn/dep-versions.git -b ${SRC_BRANCH} /usr/src/dep-versions && \
    cd /usr/src/dep-versions && \
    if [ -n "${SRC_TAG}" ] && git show-ref --tags ${SRC_TAG} > /dev/null 2>&1; then \
        echo "Checking out tag ${SRC_TAG}"; \
        cd /usr/src/dep-versions && git checkout tags/${SRC_TAG}; \
    fi

# Build liblonghorn
RUN export REPO_OVERRIDE="" && \
    export COMMIT_ID_OVERRIDE="" && \
    bash /usr/src/dep-versions/scripts/build-liblonghorn.sh "${REPO_OVERRIDE}" "${COMMIT_ID_OVERRIDE}"

# Build TGT
RUN export REPO_OVERRIDE="" && \
    export COMMIT_ID_OVERRIDE="" && \
    bash /usr/src/dep-versions/scripts/build-tgt.sh "${REPO_OVERRIDE}" "${COMMIT_ID_OVERRIDE}"

# Build spdk
RUN export REPO_OVERRIDE="" && \
    export COMMIT_ID_OVERRIDE="" && \
    bash /usr/src/dep-versions/scripts/build-spdk.sh "${REPO_OVERRIDE}" "${COMMIT_ID_OVERRIDE}" "${ARCH}"

# Build libjson-c-devel
RUN export REPO_OVERRIDE="" && \
    export COMMIT_ID_OVERRIDE="" && \
    bash /usr/src/dep-versions/scripts/build-libjsonc.sh "${REPO_OVERRIDE}" "${COMMIT_ID_OVERRIDE}"

# Build nvme-cli
RUN export REPO_OVERRIDE="" && \
    export COMMIT_ID_OVERRIDE="" && \
    bash /usr/src/dep-versions/scripts/build-nvme-cli.sh "${REPO_OVERRIDE}" "${COMMIT_ID_OVERRIDE}"

# Stage 3: copy binaries to release image
FROM registry.suse.com/bci/bci-base:15.7 AS release

ARG ARCH=amd64

## Install runtime dependencies from zypper
RUN zypper -n ref && \
    zypper update -y

RUN zypper -n install kmod jq util-linux procps awk qemu-tools e2fsprogs xfsprogs python311-base device-mapper netcat fuse3-devel logrotate libcmocka-devel

RUN ln -sf /usr/bin/python3.11 /usr/local/bin/python && \
    ln -sf /usr/bin/python3.11 /usr/local/bin/python3

RUN zypper -n install cifs-utils
RUN zypper -n install sg3_utils iproute2
RUN zypper -n install util-linux-systemd nfs-client nfs4-acl-tools
RUN zypper clean -a

# Install SPDK dependencies
COPY --from=cbuilder /usr/src/spdk/scripts /usr/src/spdk/scripts
COPY --from=cbuilder /usr/src/spdk/include /usr/src/spdk/include
RUN for i in {1..10}; do \
        bash /usr/src/spdk/scripts/pkgdep.sh && break || sleep 1; \
    done

# Copy pre-built binaries from cbuilder and gobuilder
COPY --from=gobuilder \
    /usr/local/bin/grpc_health_probe \
    /usr/local/bin/go-spdk-helper \
    /usr/local/bin/

COPY --from=cbuilder \
    /usr/local/bin/spdk_* \
    /usr/local/bin/

COPY --from=cbuilder \
    /usr/local/sbin/nvme \
    /usr/local/sbin/

COPY --from=cbuilder \
    /usr/sbin/tgt-admin \
    /usr/sbin/tgt-setup-lun \
    /usr/sbin/tgtadm \
    /usr/sbin/tgtd \
    /usr/sbin/tgtimg \
    /usr/sbin/

COPY --from=cbuilder \
   /usr/local/lib64 \
   /usr/local/lib64

COPY --from=cbuilder \
   /usr/lib64 \
   /usr/lib64

# longhorn/longhorn#11254: Clean up unnecessary *.exe that leads to a false alarm in vulnerability scanning
RUN find /usr/lib64 -depth -type f -name 'wininst-*.exe' -exec rm -f {} \;

RUN ldconfig

COPY bin/longhorn-instance-manager /usr/local/bin/
COPY package/instance-manager /usr/local/bin/
COPY package/instance-manager-v2-prestop /usr/local/bin/
COPY package/exec-logrotate /usr/local/bin/

# Verify the dependencies for the binaries
RUN ldd /usr/local/bin/* /usr/local/sbin/* /usr/sbin/* | grep "not found" && exit 1 || true

# Add Tini
ENV TINI_VERSION v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-${ARCH} /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]

CMD ["longhorn"]
</file>

<file path="package/exec-logrotate">
#!/bin/bash

function create_logrotate_config() {
    cat <<EOF > /etc/logrotate.conf
/log/spdk_tgt.log {
    rotate 2
    size 20M
    missingok
    notifempty
    copytruncate
}

/log/instance-manager-${DATA_ENGINE}.log {
    rotate 2
    size 20M
    missingok
    notifempty
    copytruncate
}

/log/prestop-${DATA_ENGINE}.log {
    rotate 2
    size 20M
    missingok
    notifempty
    copytruncate
}
EOF
}

# Create logrotate configuration
create_logrotate_config

# Like cron job, run logrotate every 10 minute
while true; do
    logrotate /etc/logrotate.conf
    sleep 600
done &
</file>

<file path="package/instance-manager">
#!/bin/bash

function show_help() {
    cat <<EOF
Usage: $0 [OPTIONS]

Options:
    -s, --enable-spdk          Enable SPDK
        --spdk-interrupt-mode  Enable SPDK interrupt mode
        --spdk-memory-size     SPDK memory size in MB
        --spdk-cpumask         SPDK CPU mask
        --spdk-no-hugepage     Disable SPDK hugepage usage
    -l, --spdk-log             SPDK log level
    -h, --help                 Show this help message and exit
EOF
    exit 0
}

function bind_dev() {
    mount --rbind /host/dev /dev
}

function bind_sys() {
    mount --rbind /host/sys /sys
}

function bind_lib_modules() {
    mount --rbind /host/lib/modules /lib/modules
}

function enable_tgtd() {
    echo "Enabling tgtd"
    tgtd -f 2>&1 | tee /var/log/tgtd.log &
}

function generate_nvme_hostid_and_hostnqn() {
    mkdir -p /etc/nvme

    local hostnqn=$(nvme gen-hostnqn)
    # hostnqn is generated from /sys/class/dmi/id/product_uuid according to the implementation of libnvme
    echo "$hostnqn" > /etc/nvme/hostnqn
    # Always generate the same hostid for the same hostnqn
    cat /sys/class/dmi/id/product_uuid > /etc/nvme/hostid
}

function enable_spdk_tgt() {
    local options=("$@")
    echo "Enabling spdk_tgt with options: ${options[*]}"
    spdk_tgt "${options[@]}" 2>&1 | tee -a /log/spdk_tgt.log &

    timeout=120  # Timeout in seconds
    interval=1  # Interval in seconds
    elapsed_time=0

    while [ $elapsed_time -lt $timeout ]; do
        if [ -S "/var/tmp/spdk.sock" ]; then
            echo "Socket file '/var/tmp/spdk.sock' found after $elapsed_time seconds."
            return 0  # Exit successfully if the file exists
        fi

        sleep $interval
        elapsed_time=$((elapsed_time + interval))
    done

    echo "Timeout reached. Socket file '/var/tmp/spdk.sock' not found."
    return 1
}

enable_spdk=0
spdk_options=()
instance_manager_options=()

while [[ $# -gt 0 ]]; do
    opt="$1"
    case $opt in
        -s|--enable-spdk)
            enable_spdk=1
            ;;
        -m|--spdk-cpumask)
            spdk_options+=("-m" "$2")
            shift
            ;;
        -s|--spdk-memory-size)
            spdk_options+=("--mem-size" "$2")
            shift
            ;;
        -g|--spdk-no-hugepage)
            spdk_options+=("--no-huge")
            ;;
        -l|--spdk-log)
            spdk_options+=("-L" "$2")
            shift
            ;;
        --spdk-interrupt-mode)
            spdk_options+=("--interrupt-mode")
            ;;
        -h|--help)
            show_help
            ;;
        *)
            instance_manager_options+=("$1")
            ;;
    esac
    shift
done

#### Main ####

if [ "$enable_spdk" -eq 1 ]; then
    mkdir -p /log
    touch /log/instance-manager-${DATA_ENGINE}.log
    touch /log/spdk_tgt.log
fi

bind_dev
bind_sys
bind_lib_modules
[ "$enable_spdk" -eq 0 ] && enable_tgtd
[ "$enable_spdk" -eq 1 ] && generate_nvme_hostid_and_hostnqn
[ "$enable_spdk" -eq 1 ] && enable_spdk_tgt "${spdk_options[@]}"

if [ "$enable_spdk" -eq 1 ]; then
    exec-logrotate
    exec longhorn-instance-manager "${instance_manager_options[@]}" 2>&1 | tee -a /log/instance-manager-${DATA_ENGINE}.log
else
    exec longhorn-instance-manager "${instance_manager_options[@]}"
fi
</file>

<file path="package/instance-manager-v2-prestop">
#!/bin/bash

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S.%N') $*"
}

function cleanup_spdk_resources() {
    log "Caught termination signal. Cleaning up SPDK resources..."

    local v2_devices=($(get_v2_devices))
    for device in "${v2_devices[@]}"; do
        dmsetup remove "$device"
        log "Removed device-mapper device: $device"

        device_file="/host/dev/longhorn/$device"
        rm "$device_file"
        log "Removed device file: $device_file"
    done

    log "Sending SIGTERM to go-spdk-helper to stop spdk_tgt..."
    go-spdk-helper kill-instance --sig-name SIGTERM
    if [[ $? -ne 0 ]]; then
        log "Failed to send SIGTERM to go-spdk-helper"
    else
        log "Successfully sent SIGTERM to go-spdk-helper"
    fi
}

function get_v2_devices() {
    local dm_devices
    dm_devices=$(dmsetup ls 2>/dev/null | awk '{print $1}')
    local v2_devices=()

    if [[ ! -d /host/dev/longhorn/ ]]; then
        echo "${v2_devices[@]}"
        return
    fi

    local device_files
    device_files=$(ls /host/dev/longhorn/)

    for dm_device in $dm_devices; do
        for device_file in $device_files; do
            if [[ "$dm_device" == "$device_file" ]]; then
                v2_devices+=("$dm_device")
                break
            fi
        done
    done

    echo "${v2_devices[@]}"
}

cleanup_spdk_resources
</file>

<file path="pkg/api/backing_image.go">
package api

import (
	"fmt"

	"github.com/sirupsen/logrus"
	"google.golang.org/protobuf/types/known/emptypb"

	rpc "github.com/longhorn/types/pkg/generated/imrpc"
)

type BackingImage struct {
	Name             string `json:"name"`
	BackingImageUUID string `json:"backing_image_uuid"`
	DiskUUID         string `json:"disk_uuid"`
	Size             uint64 `json:"size"`
	ExpectedChecksum string `json:"expected_checksum"`

	Status BackingImageStatus `json:"status"`
}

type BackingImageStatus struct {
	Progress        int    `json:"progress"`
	State           string `json:"state"`
	CurrentChecksum string `json:"currentChecksum"`
	ErrorMsg        string `json:"errorMsg"`
}

func RPCToBackingImage(obj *rpc.SPDKBackingImageResponse) (*BackingImage, error) {
	if obj == nil {
		return nil, fmt.Errorf("cannot convert nil SPDKBackingImageResponse")
	}
	if obj.Spec == nil {
		return nil, fmt.Errorf("backing image spec is nil")
	}
	bi := &BackingImage{
		Name:             obj.Spec.Name,
		BackingImageUUID: obj.Spec.BackingImageUuid,
		DiskUUID:         obj.Spec.DiskUuid,
		Size:             obj.Spec.Size,
		ExpectedChecksum: obj.Spec.Checksum,

		Status: BackingImageStatus{
			Progress:        int(obj.Status.Progress),
			State:           obj.Status.State,
			CurrentChecksum: obj.Status.Checksum,
			ErrorMsg:        obj.Status.ErrorMsg,
		},
	}

	return bi, nil
}

func RPCToBackingImageList(obj *rpc.SPDKBackingImageListResponse) map[string]*BackingImage {
	ret := map[string]*BackingImage{}
	for name, bi := range obj.BackingImages {
		res, err := RPCToBackingImage(bi)
		if err != nil {
			logrus.WithError(err).Warnf("failed to convert backing image %v", name)
		}
		ret[name] = res
	}
	return ret
}

type BackingImageStream struct {
	stream rpc.ProxyEngineService_SPDKBackingImageWatchClient
}

func NewBackingImageStream(stream rpc.ProxyEngineService_SPDKBackingImageWatchClient) *BackingImageStream {
	return &BackingImageStream{
		stream,
	}
}

func (s *BackingImageStream) Recv() (*emptypb.Empty, error) {
	return s.stream.Recv()
}
</file>

<file path="pkg/api/disk.go">
package api

type DiskInfo struct {
	ID          string
	Name        string
	UUID        string
	Path        string
	Type        string
	Driver      string
	TotalSize   int64
	FreeSize    int64
	TotalBlocks int64
	FreeBlocks  int64
	BlockSize   int64
	ClusterSize int64
	State       string
}

// ReplicaStorageInstance is utilized to represent a replica directory of a legacy volume and
// a replica logical volume (lvol) of a SPDK volume.
type ReplicaStorageInstance struct {
	Name       string
	UUID       string
	DiskName   string
	DiskUUID   string
	SpecSize   uint64
	ActualSize uint64
}

type DiskMetrics struct {
	ReadThroughput  uint64
	WriteThroughput uint64
	ReadLatency     uint64
	WriteLatency    uint64
	ReadIOPS        uint64
	WriteIOPS       uint64
}

type DiskHealth struct {
	ModelNumber                             string
	SerialNumber                            string
	FirmwareRevision                        string
	Traddr                                  string
	CriticalWarning                         uint32
	TemperatureCelsius                      float64
	AvailableSparePercentage                uint32
	AvailableSpareThresholdPercentage       uint32
	PercentageUsed                          uint32
	DataUnitsRead                           uint64
	DataUnitsWritten                        uint64
	HostReadCommands                        uint64
	HostWriteCommands                       uint64
	ControllerBusyTime                      uint64
	PowerCycles                             uint64
	PowerOnHours                            uint64
	UnsafeShutdowns                         uint64
	MediaErrors                             uint64
	NumErrLogEntries                        uint64
	WarningTemperatureTimeMinutes           uint64
	CriticalCompositeTemperatureTimeMinutes uint64
}
</file>

<file path="pkg/api/instance.go">
package api

import (
	rpc "github.com/longhorn/types/pkg/generated/imrpc"
	"github.com/longhorn/types/pkg/generated/spdkrpc"
	"google.golang.org/protobuf/types/known/emptypb"
)

var (
	dataEngines = map[string]string{
		"DATA_ENGINE_V1": "v1",
		"DATA_ENGINE_V2": "v2",
	}
)

type InstanceProcessSpec struct {
	Binary string   `json:"binary"`
	Args   []string `json:"args"`
}

type Instance struct {
	Name           string         `json:"name"`
	Type           string         `json:"type"`
	DataEngine     string         `json:"dataEngine"`
	PortCount      int32          `json:"portCount"`
	PortArgs       []string       `json:"portArgs"`
	InstanceStatus InstanceStatus `json:"instanceStatus"`
	Deleted        bool           `json:"deleted"`

	InstanceProccessSpec *InstanceProcessSpec

	// Deprecated: replaced by DataEngine.
	BackendStoreDriver string `json:"backendStoreDriver"`
}

func RPCToInstance(obj *rpc.InstanceResponse) *Instance {
	instance := &Instance{
		Name: obj.Spec.Name,
		Type: obj.Spec.Type,
		//lint:ignore SA1019 replaced with DataEngine
		BackendStoreDriver: obj.Spec.BackendStoreDriver.String(), // nolint: staticcheck
		DataEngine:         dataEngines[obj.Spec.DataEngine.String()],
		PortCount:          obj.Spec.PortCount,
		PortArgs:           obj.Spec.PortArgs,
		InstanceStatus:     RPCToInstanceStatus(obj.Status),
	}

	if obj.Spec.ProcessInstanceSpec != nil {
		instance.InstanceProccessSpec = &InstanceProcessSpec{
			Binary: obj.Spec.ProcessInstanceSpec.Binary,
			Args:   obj.Spec.ProcessInstanceSpec.Args,
		}
	}

	return instance
}

func RPCToInstanceList(obj *rpc.InstanceListResponse) map[string]*Instance {
	ret := map[string]*Instance{}
	for name, p := range obj.Instances {
		ret[name] = RPCToInstance(p)
	}
	return ret
}

type InstanceStatus struct {
	State                  string          `json:"state"`
	ErrorMsg               string          `json:"errorMsg"`
	Conditions             map[string]bool `json:"conditions"`
	PortStart              int32           `json:"portStart"`
	PortEnd                int32           `json:"portEnd"`
	TargetPortStart        int32           `json:"targetPortStart"`
	TargetPortEnd          int32           `json:"targetPortEnd"`
	StandbyTargetPortStart int32           `json:"standbyTargetPortStart"`
	StandbyTargetPortEnd   int32           `json:"standbyTargetPortEnd"`
	UblkID                 int32           `json:"ublk_id"`
	UUID                   string          `json:"uuid"`
}

func RPCToInstanceStatus(obj *rpc.InstanceStatus) InstanceStatus {
	return InstanceStatus{
		State:                  obj.State,
		ErrorMsg:               obj.ErrorMsg,
		Conditions:             obj.Conditions,
		PortStart:              obj.PortStart,
		PortEnd:                obj.PortEnd,
		TargetPortStart:        obj.TargetPortStart,
		TargetPortEnd:          obj.TargetPortEnd,
		StandbyTargetPortStart: obj.StandbyTargetPortStart,
		StandbyTargetPortEnd:   obj.StandbyTargetPortEnd,
		UblkID:                 obj.UblkId,
		UUID:                   obj.Uuid,
	}
}

type InstanceStream struct {
	stream rpc.InstanceService_InstanceWatchClient
}

func NewInstanceStream(stream rpc.InstanceService_InstanceWatchClient) *InstanceStream {
	return &InstanceStream{
		stream,
	}
}

func (s *InstanceStream) Recv() (*emptypb.Empty, error) {
	return s.stream.Recv()
}

type ReplicaStream struct {
	stream spdkrpc.SPDKService_ReplicaWatchClient
}

func NewReplicaStream(stream spdkrpc.SPDKService_ReplicaWatchClient) *ReplicaStream {
	return &ReplicaStream{
		stream,
	}
}

func (s *ReplicaStream) Recv() (*emptypb.Empty, error) {
	return s.stream.Recv()
}

type EngineStream struct {
	stream spdkrpc.SPDKService_EngineWatchClient
}

func NewEngineStream(stream spdkrpc.SPDKService_EngineWatchClient) *EngineStream {
	return &EngineStream{
		stream,
	}
}

func (s *EngineStream) Recv() (*emptypb.Empty, error) {
	return s.stream.Recv()
}

func NewLogStream(stream rpc.ProcessManagerService_ProcessLogClient) *LogStream {
	return &LogStream{
		stream,
	}
}

type LogStream struct {
	stream rpc.ProcessManagerService_ProcessLogClient
}

func (s *LogStream) Recv() (string, error) {
	resp, err := s.stream.Recv()
	if err != nil {
		return "", err
	}
	return resp.Line, nil
}
</file>

<file path="pkg/api/process.go">
package api

import (
	rpc "github.com/longhorn/types/pkg/generated/imrpc"
)

type Process struct {
	Name      string   `json:"name"`
	Binary    string   `json:"binary"`
	Args      []string `json:"args"`
	PortCount int32    `json:"portCount"`
	PortArgs  []string `json:"portArgs"`

	ProcessStatus ProcessStatus `json:"processStatus"`

	Deleted bool `json:"deleted"`
}

func RPCToProcess(obj *rpc.ProcessResponse) *Process {
	return &Process{
		Name:          obj.Spec.Name,
		Binary:        obj.Spec.Binary,
		Args:          obj.Spec.Args,
		PortCount:     obj.Spec.PortCount,
		PortArgs:      obj.Spec.PortArgs,
		ProcessStatus: RPCToProcessStatus(obj.Status),
	}
}

func RPCToProcessList(obj *rpc.ProcessListResponse) map[string]*Process {
	ret := map[string]*Process{}
	for name, p := range obj.Processes {
		ret[name] = RPCToProcess(p)
	}
	return ret
}

type ProcessStatus struct {
	State      string          `json:"state"`
	ErrorMsg   string          `json:"errorMsg"`
	Conditions map[string]bool `json:"conditions"`
	PortStart  int32           `json:"portStart"`
	PortEnd    int32           `json:"portEnd"`
	UUID       string          `json:"uuid"`
}

func RPCToProcessStatus(obj *rpc.ProcessStatus) ProcessStatus {
	return ProcessStatus{
		State:      obj.State,
		ErrorMsg:   obj.ErrorMsg,
		Conditions: obj.Conditions,
		PortStart:  obj.PortStart,
		PortEnd:    obj.PortEnd,
		UUID:       obj.Uuid,
	}
}

type ProcessStream struct {
	stream rpc.ProcessManagerService_ProcessWatchClient
}

func NewProcessStream(stream rpc.ProcessManagerService_ProcessWatchClient) *ProcessStream {
	return &ProcessStream{
		stream,
	}
}

func (s *ProcessStream) Recv() (*rpc.ProcessResponse, error) {
	return s.stream.Recv()
}
</file>

<file path="pkg/client/disk.go">
package client

import (
	"context"
	"crypto/tls"
	"fmt"

	"github.com/cockroachdb/errors"
	"google.golang.org/grpc"
	"google.golang.org/protobuf/types/known/emptypb"

	healthpb "google.golang.org/grpc/health/grpc_health_v1"

	rpc "github.com/longhorn/types/pkg/generated/imrpc"

	"github.com/longhorn/longhorn-instance-manager/pkg/api"
	"github.com/longhorn/longhorn-instance-manager/pkg/meta"
	"github.com/longhorn/longhorn-instance-manager/pkg/types"
	"github.com/longhorn/longhorn-instance-manager/pkg/util"
)

type DiskServiceContext struct {
	cc *grpc.ClientConn

	ctx  context.Context
	quit context.CancelFunc

	service rpc.DiskServiceClient
	health  healthpb.HealthClient
}

func (c *DiskServiceClient) Close() error {
	if c.cc == nil {
		return nil
	}
	return c.cc.Close()
}

func (c *DiskServiceClient) getDiskServiceClient() rpc.DiskServiceClient {
	return c.service
}

type DiskServiceClient struct {
	serviceURL string
	tlsConfig  *tls.Config
	DiskServiceContext
}

// NewDiskServiceClient creates a new DiskServiceClient.
func NewDiskServiceClient(ctx context.Context, ctxCancel context.CancelFunc, serviceURL string, tlsConfig *tls.Config) (*DiskServiceClient, error) {
	getDiskServiceContext := func(serviceUrl string, tlsConfig *tls.Config) (DiskServiceContext, error) {
		connection, err := util.Connect(serviceUrl, tlsConfig)
		if err != nil {
			return DiskServiceContext{}, errors.Wrapf(err, "cannot connect to Disk Service %v", serviceUrl)
		}

		return DiskServiceContext{
			cc:      connection,
			ctx:     ctx,
			quit:    ctxCancel,
			service: rpc.NewDiskServiceClient(connection),
			health:  healthpb.NewHealthClient(connection),
		}, nil
	}

	serviceContext, err := getDiskServiceContext(serviceURL, tlsConfig)
	if err != nil {
		return nil, err
	}

	return &DiskServiceClient{
		serviceURL:         serviceURL,
		tlsConfig:          tlsConfig,
		DiskServiceContext: serviceContext,
	}, nil
}

// NewDiskServiceClientWithTLS creates a new DiskServiceClient with TLS
func NewDiskServiceClientWithTLS(ctx context.Context, ctxCancel context.CancelFunc, serviceURL, caFile, certFile, keyFile, peerName string) (*DiskServiceClient, error) {
	tlsConfig, err := util.LoadClientTLS(caFile, certFile, keyFile, peerName)
	if err != nil {
		return nil, errors.Wrap(err, "failed to load tls key pair from file")
	}

	return NewDiskServiceClient(ctx, ctxCancel, serviceURL, tlsConfig)
}

// DiskCreate creates a disk with the given name and path.
// diskUUID is optional, if not provided, it indicates the disk is newly added.
func (c *DiskServiceClient) DiskCreate(diskType, diskName, diskUUID, diskPath, diskDriver string, blockSize int64) (*api.DiskInfo, error) {
	if diskName == "" || diskPath == "" {
		return nil, fmt.Errorf("failed to create disk: missing required parameters")
	}

	t, ok := rpc.DiskType_value[diskType]
	if !ok {
		return nil, fmt.Errorf("failed to get disk info: invalid disk type %v", diskType)
	}

	client := c.getDiskServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	resp, err := client.DiskCreate(ctx, &rpc.DiskCreateRequest{
		DiskType:   rpc.DiskType(t),
		DiskName:   diskName,
		DiskUuid:   diskUUID,
		DiskPath:   diskPath,
		BlockSize:  blockSize,
		DiskDriver: diskDriver,
	})
	if err != nil {
		return nil, err
	}

	return &api.DiskInfo{
		ID:          resp.GetId(),
		Name:        resp.GetName(),
		UUID:        resp.GetUuid(),
		Path:        resp.GetPath(),
		Type:        resp.GetType(),
		Driver:      resp.GetDriver(),
		TotalSize:   resp.GetTotalSize(),
		FreeSize:    resp.GetFreeSize(),
		TotalBlocks: resp.GetTotalBlocks(),
		FreeBlocks:  resp.GetFreeBlocks(),
		BlockSize:   resp.GetBlockSize(),
		ClusterSize: resp.GetClusterSize(),
		State:       resp.GetState(),
	}, nil
}

// DiskGet returns the disk info with the given name and path.
func (c *DiskServiceClient) DiskGet(diskType, diskName, diskPath, diskDriver string) (*api.DiskInfo, error) {
	if diskName == "" {
		return nil, fmt.Errorf("failed to get disk info: missing required parameter diskName")
	}

	t, ok := rpc.DiskType_value[diskType]
	if !ok {
		return nil, fmt.Errorf("failed to get disk info: invalid disk type %v", diskType)
	}

	client := c.getDiskServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	resp, err := client.DiskGet(ctx, &rpc.DiskGetRequest{
		DiskType:   rpc.DiskType(t),
		DiskName:   diskName,
		DiskPath:   diskPath,
		DiskDriver: diskDriver,
	})
	if err != nil {
		return nil, err
	}

	return &api.DiskInfo{
		ID:          resp.GetId(),
		Name:        resp.GetName(),
		UUID:        resp.GetUuid(),
		Path:        resp.GetPath(),
		Type:        resp.GetType(),
		Driver:      resp.GetDriver(),
		TotalSize:   resp.GetTotalSize(),
		FreeSize:    resp.GetFreeSize(),
		TotalBlocks: resp.GetTotalBlocks(),
		FreeBlocks:  resp.GetFreeBlocks(),
		BlockSize:   resp.GetBlockSize(),
		ClusterSize: resp.GetClusterSize(),
		State:       resp.GetState(),
	}, nil
}

// DiskHealthGet returns the disk health info with the given name, path and driver.
func (c *DiskServiceClient) DiskHealthGet(diskType, diskName, diskPath, diskDriver string) (*api.DiskHealth, error) {
	if diskName == "" {
		return nil, fmt.Errorf("failed to get disk health info: missing required parameter diskName")
	}

	t, ok := rpc.DiskType_value[diskType]
	if !ok {
		return nil, fmt.Errorf("failed to get disk health info: invalid disk type %v", diskType)
	}

	client := c.getDiskServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	resp, err := client.DiskHealthGet(ctx, &rpc.DiskHealthGetRequest{
		DiskType:   rpc.DiskType(t),
		DiskName:   diskName,
		DiskPath:   diskPath,
		DiskDriver: diskDriver,
	})
	if err != nil {
		return nil, err
	}

	return &api.DiskHealth{
		ModelNumber:                             resp.GetModelNumber(),
		SerialNumber:                            resp.GetSerialNumber(),
		FirmwareRevision:                        resp.GetFirmwareRevision(),
		Traddr:                                  resp.GetTraddr(),
		CriticalWarning:                         resp.GetCriticalWarning(),
		TemperatureCelsius:                      resp.GetTemperatureCelsius(),
		AvailableSparePercentage:                resp.GetAvailableSparePercentage(),
		AvailableSpareThresholdPercentage:       resp.GetAvailableSpareThresholdPercentage(),
		PercentageUsed:                          resp.GetPercentageUsed(),
		DataUnitsRead:                           resp.GetDataUnitsRead(),
		DataUnitsWritten:                        resp.GetDataUnitsWritten(),
		HostReadCommands:                        resp.GetHostReadCommands(),
		HostWriteCommands:                       resp.GetHostWriteCommands(),
		ControllerBusyTime:                      resp.GetControllerBusyTime(),
		PowerCycles:                             resp.GetPowerCycles(),
		PowerOnHours:                            resp.GetPowerOnHours(),
		UnsafeShutdowns:                         resp.GetUnsafeShutdowns(),
		MediaErrors:                             resp.GetMediaErrors(),
		NumErrLogEntries:                        resp.GetNumErrLogEntries(),
		WarningTemperatureTimeMinutes:           resp.GetWarningTemperatureTimeMinutes(),
		CriticalCompositeTemperatureTimeMinutes: resp.GetCriticalCompositeTemperatureTimeMinutes(),
	}, nil
}

// DiskDelete deletes the disk with the given name, disk name, disk UUID, disk path and disk driver.
func (c *DiskServiceClient) DiskDelete(diskType, diskName, diskUUID, diskPath, diskDriver string) error {
	if diskName == "" {
		return fmt.Errorf("failed to delete disk: missing required diskName")
	}

	client := c.getDiskServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	_, err := client.DiskDelete(ctx, &rpc.DiskDeleteRequest{
		DiskType:   rpc.DiskType(rpc.DiskType_value[diskType]),
		DiskName:   diskName,
		DiskUuid:   diskUUID,
		DiskPath:   diskPath,
		DiskDriver: diskDriver,
	})
	return err
}

func (c *DiskServiceClient) DiskReplicaInstanceList(diskType, diskName, diskDriver string) (map[string]*api.ReplicaStorageInstance, error) {
	if diskName == "" {
		return nil, fmt.Errorf("failed to list replica instances on disk: missing required parameter")
	}

	client := c.getDiskServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	resp, err := client.DiskReplicaInstanceList(ctx, &rpc.DiskReplicaInstanceListRequest{
		DiskType:   rpc.DiskType(rpc.DiskType_value[diskType]),
		DiskName:   diskName,
		DiskDriver: diskDriver,
	})
	if err != nil {
		return nil, err
	}

	instances := map[string]*api.ReplicaStorageInstance{}
	for name, instance := range resp.ReplicaInstances {
		instances[name] = &api.ReplicaStorageInstance{
			Name:       instance.Name,
			UUID:       instance.Uuid,
			DiskName:   instance.DiskName,
			DiskUUID:   instance.DiskUuid,
			SpecSize:   instance.SpecSize,
			ActualSize: instance.ActualSize,
		}
	}

	return instances, nil
}

// DiskReplicaInstanceDelete deletes the replica instance with the given name on the disk.
func (c *DiskServiceClient) DiskReplicaInstanceDelete(diskType, diskName, diskUUID, diskDriver, replciaInstanceName string) error {
	if diskName == "" || diskUUID == "" || replciaInstanceName == "" {
		return fmt.Errorf("failed to delete replica instance on disk: missing required parameters")
	}

	client := c.getDiskServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	_, err := client.DiskReplicaInstanceDelete(ctx, &rpc.DiskReplicaInstanceDeleteRequest{
		DiskType:            rpc.DiskType(rpc.DiskType_value[diskType]),
		DiskName:            diskName,
		DiskUuid:            diskUUID,
		DiskDriver:          diskDriver,
		ReplciaInstanceName: replciaInstanceName,
	})

	return err
}

// VersionGet returns the disk service version.
func (c *DiskServiceClient) VersionGet() (*meta.DiskServiceVersionOutput, error) {
	client := c.getDiskServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	resp, err := client.VersionGet(ctx, &emptypb.Empty{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to get disk service version")
	}

	return &meta.DiskServiceVersionOutput{
		Version:   resp.Version,
		GitCommit: resp.GitCommit,
		BuildDate: resp.BuildDate,

		InstanceManagerDiskServiceAPIVersion:    int(resp.InstanceManagerDiskServiceAPIVersion),
		InstanceManagerDiskServiceAPIMinVersion: int(resp.InstanceManagerDiskServiceAPIMinVersion),
	}, nil
}

func (c *DiskServiceClient) CheckConnection() error {
	req := &healthpb.HealthCheckRequest{}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err := c.health.Check(ctx, req)
	return err
}

// MetricsGet returns the disk metrics with the given name and path.
func (c *DiskServiceClient) MetricsGet(diskType, diskName, diskPath, diskDriver string) (*api.DiskMetrics, error) {
	if diskName == "" {
		return nil, fmt.Errorf("failed to get disk metrics: missing required parameter diskName")
	}

	t, ok := rpc.DiskType_value[diskType]
	if !ok {
		return nil, fmt.Errorf("failed to get disk metrics: invalid disk type %v", diskType)
	}

	client := c.getDiskServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	resp, err := client.MetricsGet(ctx, &rpc.DiskGetRequest{
		DiskType:   rpc.DiskType(t),
		DiskName:   diskName,
		DiskPath:   diskPath,
		DiskDriver: diskDriver,
	})
	if err != nil {
		return nil, err
	}

	// Convert to api.DiskMetrics format
	return &api.DiskMetrics{
		ReadThroughput:  resp.Metrics.ReadThroughput,
		WriteThroughput: resp.Metrics.WriteThroughput,
		ReadLatency:     resp.Metrics.ReadLatency,
		WriteLatency:    resp.Metrics.WriteLatency,
		ReadIOPS:        resp.Metrics.ReadIOPS,
		WriteIOPS:       resp.Metrics.WriteIOPS,
	}, nil
}
</file>

<file path="pkg/client/instance.go">
package client

import (
	"context"
	"crypto/tls"
	"fmt"

	"github.com/cockroachdb/errors"
	"google.golang.org/grpc"
	"google.golang.org/protobuf/types/known/emptypb"

	healthpb "google.golang.org/grpc/health/grpc_health_v1"

	rpc "github.com/longhorn/types/pkg/generated/imrpc"

	"github.com/longhorn/longhorn-instance-manager/pkg/api"
	"github.com/longhorn/longhorn-instance-manager/pkg/meta"
	"github.com/longhorn/longhorn-instance-manager/pkg/types"
	"github.com/longhorn/longhorn-instance-manager/pkg/util"
)

type InstanceServiceContext struct {
	cc *grpc.ClientConn

	ctx  context.Context
	quit context.CancelFunc

	service rpc.InstanceServiceClient
	health  healthpb.HealthClient
}

func (c InstanceServiceContext) Close() error {
	c.quit()
	if c.cc == nil {
		return nil
	}
	if err := c.cc.Close(); err != nil {
		return errors.Wrap(err, "failed to close instance gRPC connection")
	}
	return nil
}

func (c *InstanceServiceClient) getControllerServiceClient() rpc.InstanceServiceClient {
	return c.service
}

type InstanceServiceClient struct {
	serviceURL string
	tlsConfig  *tls.Config
	InstanceServiceContext
}

func NewInstanceServiceClient(ctx context.Context, ctxCancel context.CancelFunc, serviceURL string, tlsConfig *tls.Config) (*InstanceServiceClient, error) {
	getInstanceServiceContext := func(serviceUrl string, tlsConfig *tls.Config) (InstanceServiceContext, error) {
		connection, err := util.Connect(serviceUrl, tlsConfig)
		if err != nil {
			return InstanceServiceContext{}, errors.Wrapf(err, "cannot connect to Instance Service %v", serviceUrl)
		}

		return InstanceServiceContext{
			cc:      connection,
			ctx:     ctx,
			quit:    ctxCancel,
			service: rpc.NewInstanceServiceClient(connection),
			health:  healthpb.NewHealthClient(connection),
		}, nil
	}

	serviceContext, err := getInstanceServiceContext(serviceURL, tlsConfig)
	if err != nil {
		return nil, err
	}

	return &InstanceServiceClient{
		serviceURL:             serviceURL,
		tlsConfig:              tlsConfig,
		InstanceServiceContext: serviceContext,
	}, nil
}

func NewInstanceServiceClientWithTLS(ctx context.Context, ctxCancel context.CancelFunc, serviceURL, caFile, certFile, keyFile, peerName string) (*InstanceServiceClient, error) {
	tlsConfig, err := util.LoadClientTLS(caFile, certFile, keyFile, peerName)
	if err != nil {
		return nil, errors.Wrap(err, "failed to load tls key pair from file")
	}

	return NewInstanceServiceClient(ctx, ctxCancel, serviceURL, tlsConfig)
}

type EngineCreateRequest struct {
	ReplicaAddressMap map[string]string
	Frontend          string
	UblkQueueDepth    int
	UblkNumberOfQueue int
	InitiatorAddress  string
	TargetAddress     string
	UpgradeRequired   bool
	SalvageRequested  bool
}

type ReplicaCreateRequest struct {
	DiskName         string
	DiskUUID         string
	ExposeRequired   bool
	BackingImageName string
}

type InstanceCreateRequest struct {
	DataEngine   string
	Name         string
	InstanceType string
	VolumeName   string
	Size         uint64
	PortCount    int
	PortArgs     []string

	Binary     string
	BinaryArgs []string

	Engine  EngineCreateRequest
	Replica ReplicaCreateRequest

	// Deprecated: replaced by DataEngine.
	BackendStoreDriver string
}

// InstanceCreate creates an instance.
func (c *InstanceServiceClient) InstanceCreate(req *InstanceCreateRequest) (*api.Instance, error) {
	if req.Name == "" || req.InstanceType == "" {
		return nil, fmt.Errorf("failed to create instance: missing required parameter")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(req.DataEngine)]
	if !ok {
		return nil, fmt.Errorf("failed to delete instance: invalid data engine %v", req.DataEngine)
	}

	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	var processInstanceSpec *rpc.ProcessInstanceSpec
	var spdkInstanceSpec *rpc.SpdkInstanceSpec
	if rpc.DataEngine(driver) == rpc.DataEngine_DATA_ENGINE_V1 {
		processInstanceSpec = &rpc.ProcessInstanceSpec{
			Binary: req.Binary,
			Args:   req.BinaryArgs,
		}
	} else {
		switch req.InstanceType {
		case types.InstanceTypeEngine:
			spdkInstanceSpec = &rpc.SpdkInstanceSpec{
				Size:              req.Size,
				ReplicaAddressMap: req.Engine.ReplicaAddressMap,
				Frontend:          req.Engine.Frontend,
				SalvageRequested:  req.Engine.SalvageRequested,
				UblkQueueDepth:    int32(req.Engine.UblkQueueDepth),
				UblkNumberOfQueue: int32(req.Engine.UblkNumberOfQueue),
			}
		case types.InstanceTypeReplica:
			spdkInstanceSpec = &rpc.SpdkInstanceSpec{
				Size:             req.Size,
				DiskName:         req.Replica.DiskName,
				DiskUuid:         req.Replica.DiskUUID,
				ExposeRequired:   req.Replica.ExposeRequired,
				BackingImageName: req.Replica.BackingImageName,
			}
		default:
			return nil, fmt.Errorf("failed to create instance: invalid instance type %v", req.InstanceType)
		}
	}

	p, err := client.InstanceCreate(ctx, &rpc.InstanceCreateRequest{
		Spec: &rpc.InstanceSpec{
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			Name:               req.Name,
			Type:               req.InstanceType,
			VolumeName:         req.VolumeName,
			PortCount:          int32(req.PortCount),
			PortArgs:           req.PortArgs,

			ProcessInstanceSpec: processInstanceSpec,
			SpdkInstanceSpec:    spdkInstanceSpec,

			UpgradeRequired:  req.Engine.UpgradeRequired,
			InitiatorAddress: req.Engine.InitiatorAddress,
			TargetAddress:    req.Engine.TargetAddress,
		},
	})
	if err != nil {
		return nil, errors.Wrap(err, "failed to create instance")
	}

	return api.RPCToInstance(p), nil
}

// InstanceDelete deletes the instance by name. UUID will be validated if not empty.
func (c *InstanceServiceClient) InstanceDelete(dataEngine, name, uuid, instanceType, diskUUID string, cleanupRequired bool) (*api.Instance, error) {
	if name == "" {
		return nil, fmt.Errorf("failed to delete instance: missing required parameter name")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return nil, fmt.Errorf("failed to delete instance: invalid data engine %v", dataEngine)
	}

	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	p, err := client.InstanceDelete(ctx, &rpc.InstanceDeleteRequest{
		Name: name,
		Uuid: uuid,
		Type: instanceType,
		// nolint:all replaced with DataEngine
		BackendStoreDriver: rpc.BackendStoreDriver(driver),
		DataEngine:         rpc.DataEngine(driver),
		DiskUuid:           diskUUID,
		CleanupRequired:    cleanupRequired,
	})
	if err != nil {
		return nil, errors.Wrapf(err, "failed to delete instance %v", name)
	}
	return api.RPCToInstance(p), nil
}

// InstanceGet returns the instance by name.
func (c *InstanceServiceClient) InstanceGet(dataEngine, name, instanceType string) (*api.Instance, error) {
	if name == "" {
		return nil, fmt.Errorf("failed to get instance: missing required parameter name")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return nil, fmt.Errorf("failed to get instance: invalid data engine %v", dataEngine)
	}

	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	p, err := client.InstanceGet(ctx, &rpc.InstanceGetRequest{
		Name: name,
		Type: instanceType,
		// nolint:all replaced with DataEngine
		BackendStoreDriver: rpc.BackendStoreDriver(driver),
		DataEngine:         rpc.DataEngine(driver),
	})
	if err != nil {
		return nil, errors.Wrapf(err, "failed to get instance %v", name)
	}
	return api.RPCToInstance(p), nil
}

func (c *InstanceServiceClient) InstanceList() (map[string]*api.Instance, error) {
	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	instances, err := client.InstanceList(ctx, &emptypb.Empty{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to list instances")
	}
	return api.RPCToInstanceList(instances), nil
}

// InstanceLog returns the log stream of an instance.
func (c *InstanceServiceClient) InstanceLog(ctx context.Context, dataEngine, name, instanceType string) (*api.LogStream, error) {
	if name == "" {
		return nil, fmt.Errorf("failed to get instance: missing required parameter name")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return nil, fmt.Errorf("failed to log instance: invalid data engine %v", dataEngine)
	}

	client := c.getControllerServiceClient()
	stream, err := client.InstanceLog(ctx, &rpc.InstanceLogRequest{
		Name: name,
		Type: instanceType,
		// nolint:all replaced with DataEngine
		BackendStoreDriver: rpc.BackendStoreDriver(driver),
		DataEngine:         rpc.DataEngine(driver),
	})
	if err != nil {
		return nil, errors.Wrapf(err, "failed to get instance log of %v", name)
	}
	return api.NewLogStream(stream), nil
}

// InstanceWatch watches for instance updates.
func (c *InstanceServiceClient) InstanceWatch(ctx context.Context) (*api.InstanceStream, error) {
	client := c.getControllerServiceClient()
	stream, err := client.InstanceWatch(ctx, &emptypb.Empty{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to open instance update stream")
	}

	return api.NewInstanceStream(stream), nil
}

// InstanceReplace replaces an instance with a new one.
func (c *InstanceServiceClient) InstanceReplace(dataEngine, name, instanceType, binary string, portCount int, args, portArgs []string, terminateSignal string) (*api.Instance, error) {
	if name == "" || binary == "" {
		return nil, fmt.Errorf("failed to replace instance: missing required parameter")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return nil, fmt.Errorf("failed to replace instance: invalid data engine %v", dataEngine)
	}

	if terminateSignal != "SIGHUP" {
		return nil, fmt.Errorf("unsupported terminate signal %v", terminateSignal)
	}

	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	p, err := client.InstanceReplace(ctx, &rpc.InstanceReplaceRequest{
		Spec: &rpc.InstanceSpec{
			Name: name,
			Type: instanceType,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			ProcessInstanceSpec: &rpc.ProcessInstanceSpec{
				Binary: binary,
				Args:   args,
			},
			PortCount: int32(portCount),
			PortArgs:  portArgs,
		},
		TerminateSignal: terminateSignal,
	})
	if err != nil {
		return nil, errors.Wrap(err, "failed to replace instance")
	}
	return api.RPCToInstance(p), nil
}

// InstanceSuspend suspends an instance.
func (c *InstanceServiceClient) InstanceSuspend(dataEngine, name, instanceType string) error {
	if name == "" {
		return fmt.Errorf("failed to suspend instance: missing required parameter name")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to suspend instance: invalid data engine %v", dataEngine)
	}

	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	_, err := client.InstanceSuspend(ctx, &rpc.InstanceSuspendRequest{
		Name:       name,
		Type:       instanceType,
		DataEngine: rpc.DataEngine(driver),
	})
	if err != nil {
		return errors.Wrapf(err, "failed to suspend instance %v", name)
	}

	return nil
}

// InstanceResume suspends an instance.
func (c *InstanceServiceClient) InstanceResume(dataEngine, name, instanceType string) error {
	if name == "" {
		return fmt.Errorf("failed to resume instance: missing required parameter name")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to resume instance: invalid data engine %v", dataEngine)
	}

	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	_, err := client.InstanceResume(ctx, &rpc.InstanceResumeRequest{
		Name:       name,
		Type:       instanceType,
		DataEngine: rpc.DataEngine(driver),
	})
	if err != nil {
		return errors.Wrapf(err, "failed to resume instance %v", name)
	}

	return nil
}

// InstanceSwitchOverTarget switches over the target for an instance.
func (c *InstanceServiceClient) InstanceSwitchOverTarget(dataEngine, name, instanceType, targetAddress string) error {
	if name == "" {
		return fmt.Errorf("failed to switch over target for instance: missing required parameter name")
	}

	if targetAddress == "" {
		return fmt.Errorf("failed to switch over target for instance: missing required parameter target address")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to switch over target instance: invalid data engine %v", dataEngine)
	}

	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	_, err := client.InstanceSwitchOverTarget(ctx, &rpc.InstanceSwitchOverTargetRequest{
		Name:          name,
		Type:          instanceType,
		DataEngine:    rpc.DataEngine(driver),
		TargetAddress: targetAddress,
	})
	if err != nil {
		return errors.Wrapf(err, "failed to switch over target for instance %v", name)
	}

	return nil
}

// InstanceDeleteTarget delete target for an instance.
func (c *InstanceServiceClient) InstanceDeleteTarget(dataEngine, name, instanceType string) error {
	if name == "" {
		return fmt.Errorf("failed to delete target for instance: missing required parameter name")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to delete target instance: invalid data engine %v", dataEngine)
	}

	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	_, err := client.InstanceDeleteTarget(ctx, &rpc.InstanceDeleteTargetRequest{
		Name:       name,
		Type:       instanceType,
		DataEngine: rpc.DataEngine(driver),
	})
	if err != nil {
		return errors.Wrapf(err, "failed to delete target for instance %v", name)
	}

	return nil
}

// InstanceResume resumes an instance.
func (c *InstanceServiceClient) VersionGet() (*meta.VersionOutput, error) {
	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	resp, err := client.VersionGet(ctx, &emptypb.Empty{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to get version")
	}

	return &meta.VersionOutput{
		Version:   resp.Version,
		GitCommit: resp.GitCommit,
		BuildDate: resp.BuildDate,

		InstanceManagerAPIVersion:    int(resp.InstanceManagerAPIVersion),
		InstanceManagerAPIMinVersion: int(resp.InstanceManagerAPIMinVersion),

		InstanceManagerProxyAPIVersion:    int(resp.InstanceManagerProxyAPIVersion),
		InstanceManagerProxyAPIMinVersion: int(resp.InstanceManagerProxyAPIMinVersion),
	}, nil
}

// LogSetLevel sets the log level.
func (c *InstanceServiceClient) LogSetLevel(dataEngine, service, level string) error {
	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to set log level: invalid data engine %v", dataEngine)
	}

	_, err := client.LogSetLevel(ctx, &rpc.LogSetLevelRequest{
		DataEngine: rpc.DataEngine(driver),
		Level:      level,
	})
	return err
}

// LogSetFlags sets the log flags.
func (c *InstanceServiceClient) LogSetFlags(dataEngine, service, flags string) error {
	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to set log flags: invalid data engine %v", dataEngine)
	}

	_, err := client.LogSetFlags(ctx, &rpc.LogSetFlagsRequest{
		DataEngine: rpc.DataEngine(driver),
		Flags:      flags,
	})
	return err
}

// LogGetLevel returns the log level.
func (c *InstanceServiceClient) LogGetLevel(dataEngine, service string) (string, error) {
	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return "", fmt.Errorf("failed to get log level: invalid data engine %v", dataEngine)
	}

	resp, err := client.LogGetLevel(ctx, &rpc.LogGetLevelRequest{
		DataEngine: rpc.DataEngine(driver),
	})
	if err != nil {
		return "", err
	}
	return resp.Level, nil
}

// LogGetFlags returns the log flags.
func (c *InstanceServiceClient) LogGetFlags(dataEngine, service string) (string, error) {
	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return "", fmt.Errorf("failed to get log flags: invalid data engine %v", dataEngine)
	}

	resp, err := client.LogGetFlags(ctx, &rpc.LogGetFlagsRequest{
		DataEngine: rpc.DataEngine(driver),
	})
	if err != nil {
		return "", err
	}
	return resp.Flags, nil
}

func (c *InstanceServiceClient) CheckConnection() error {
	req := &healthpb.HealthCheckRequest{}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err := c.health.Check(ctx, req)
	return err
}
</file>

<file path="pkg/client/process_manager.go">
package client

import (
	"context"
	"crypto/tls"
	"fmt"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"google.golang.org/grpc"
	"google.golang.org/protobuf/types/known/emptypb"

	healthpb "google.golang.org/grpc/health/grpc_health_v1"

	rpc "github.com/longhorn/types/pkg/generated/imrpc"

	"github.com/longhorn/longhorn-instance-manager/pkg/api"
	"github.com/longhorn/longhorn-instance-manager/pkg/meta"
	"github.com/longhorn/longhorn-instance-manager/pkg/types"
	"github.com/longhorn/longhorn-instance-manager/pkg/util"
)

type ProcessManagerServiceContext struct {
	cc *grpc.ClientConn

	ctx  context.Context
	quit context.CancelFunc

	service rpc.ProcessManagerServiceClient
	health  healthpb.HealthClient
}

func (c ProcessManagerServiceContext) Close() error {
	c.quit()
	if c.cc == nil {
		return nil
	}
	if err := c.cc.Close(); err != nil {
		return errors.Wrap(err, "failed to close process manager gRPC connection")
	}
	return nil
}

func (c *ProcessManagerClient) getControllerServiceClient() rpc.ProcessManagerServiceClient {
	return c.service
}

type ProcessManagerClient struct {
	serviceURL string
	tlsConfig  *tls.Config
	ProcessManagerServiceContext
}

func NewProcessManagerClient(ctx context.Context, ctxCancel context.CancelFunc, serviceURL string, tlsConfig *tls.Config) (*ProcessManagerClient, error) {
	getProcessManagerServiceContext := func(serviceUrl string, tlsConfig *tls.Config) (ProcessManagerServiceContext, error) {
		connection, err := util.Connect(serviceUrl, tlsConfig)
		if err != nil {
			return ProcessManagerServiceContext{}, errors.Wrapf(err, "cannot connect to ProcessManagerService %v", serviceUrl)
		}

		return ProcessManagerServiceContext{
			cc:      connection,
			ctx:     ctx,
			quit:    ctxCancel,
			service: rpc.NewProcessManagerServiceClient(connection),
			health:  healthpb.NewHealthClient(connection),
		}, nil
	}

	serviceContext, err := getProcessManagerServiceContext(serviceURL, tlsConfig)
	if err != nil {
		return nil, err
	}

	return &ProcessManagerClient{
		serviceURL:                   serviceURL,
		tlsConfig:                    tlsConfig,
		ProcessManagerServiceContext: serviceContext,
	}, nil
}

func NewProcessManagerClientWithTLS(ctx context.Context, ctxCancel context.CancelFunc, serviceURL, caFile, certFile, keyFile, peerName string) (*ProcessManagerClient, error) {
	tlsConfig, err := util.LoadClientTLS(caFile, certFile, keyFile, peerName)
	if err != nil {
		return nil, errors.Wrap(err, "failed to load tls key pair from file")
	}

	return NewProcessManagerClient(ctx, ctxCancel, serviceURL, tlsConfig)
}

func (c *ProcessManagerClient) ProcessCreate(name, binary string, portCount int, args, portArgs []string) (*rpc.ProcessResponse, error) {
	logrus.WithFields(logrus.Fields{
		"name":      name,
		"binary":    binary,
		"args":      args,
		"portCount": portCount,
		"portArgs":  portArgs,
	}).Info("Creating process")

	if name == "" || binary == "" {
		return nil, fmt.Errorf("failed to start process: missing required parameter")
	}

	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	return client.ProcessCreate(ctx, &rpc.ProcessCreateRequest{
		Spec: &rpc.ProcessSpec{
			Name:      name,
			Binary:    binary,
			Args:      args,
			PortCount: int32(portCount),
			PortArgs:  portArgs,
		},
	})
}

func (c *ProcessManagerClient) ProcessDelete(name, uuid string) (*rpc.ProcessResponse, error) {
	if name == "" {
		return nil, fmt.Errorf("failed to delete process: missing required parameter name")
	}

	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	return client.ProcessDelete(ctx, &rpc.ProcessDeleteRequest{
		Name: name,
		Uuid: uuid,
	})
}

func (c *ProcessManagerClient) ProcessGet(name string) (*rpc.ProcessResponse, error) {
	if name == "" {
		return nil, fmt.Errorf("failed to get process: missing required parameter name")
	}

	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	return client.ProcessGet(ctx, &rpc.ProcessGetRequest{
		Name: name,
	})
}

func (c *ProcessManagerClient) ProcessList() (map[string]*rpc.ProcessResponse, error) {
	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	resp, err := client.ProcessList(ctx, &rpc.ProcessListRequest{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to list processes")
	}
	return resp.Processes, nil
}

func (c *ProcessManagerClient) ProcessLog(ctx context.Context, name string) (*api.LogStream, error) {
	if name == "" {
		return nil, fmt.Errorf("failed to get process: missing required parameter name")
	}

	client := c.getControllerServiceClient()
	stream, err := client.ProcessLog(ctx, &rpc.LogRequest{
		Name: name,
	})
	if err != nil {
		return nil, errors.Wrapf(err, "failed to get process log of %v", name)
	}
	return api.NewLogStream(stream), nil
}

func (c *ProcessManagerClient) ProcessWatch(ctx context.Context) (*api.ProcessStream, error) {
	client := c.getControllerServiceClient()
	stream, err := client.ProcessWatch(ctx, &emptypb.Empty{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to open process update stream")
	}

	return api.NewProcessStream(stream), nil
}

func (c *ProcessManagerClient) ProcessReplace(name, binary string, portCount int, args, portArgs []string, terminateSignal string) (*rpc.ProcessResponse, error) {
	if name == "" || binary == "" {
		return nil, fmt.Errorf("failed to start process: missing required parameter")
	}
	if terminateSignal != "SIGHUP" {
		return nil, fmt.Errorf("unsupported terminate signal %v", terminateSignal)
	}

	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	return client.ProcessReplace(ctx, &rpc.ProcessReplaceRequest{
		Spec: &rpc.ProcessSpec{
			Name:      name,
			Binary:    binary,
			Args:      args,
			PortCount: int32(portCount),
			PortArgs:  portArgs,
		},
		TerminateSignal: terminateSignal,
	})
}

func (c *ProcessManagerClient) VersionGet() (*meta.VersionOutput, error) {

	client := c.getControllerServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	resp, err := client.VersionGet(ctx, &emptypb.Empty{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to get version")
	}

	return &meta.VersionOutput{
		Version:   resp.Version,
		GitCommit: resp.GitCommit,
		BuildDate: resp.BuildDate,

		InstanceManagerAPIVersion:    int(resp.InstanceManagerAPIVersion),
		InstanceManagerAPIMinVersion: int(resp.InstanceManagerAPIMinVersion),

		InstanceManagerProxyAPIVersion:    int(resp.InstanceManagerProxyAPIVersion),
		InstanceManagerProxyAPIMinVersion: int(resp.InstanceManagerProxyAPIMinVersion),
	}, nil
}

func (c *ProcessManagerClient) CheckConnection() error {
	req := &healthpb.HealthCheckRequest{}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err := c.health.Check(ctx, req)
	return err
}
</file>

<file path="pkg/client/proxy_backing_image.go">
package client

import (
	"context"
	"fmt"

	"github.com/cockroachdb/errors"
	"google.golang.org/protobuf/types/known/emptypb"

	"github.com/longhorn/longhorn-instance-manager/pkg/api"
	"github.com/longhorn/longhorn-instance-manager/pkg/types"

	rpc "github.com/longhorn/types/pkg/generated/imrpc"
)

func (c *ProxyClient) SPDKBackingImageCreate(name, backingImageUUID, diskUUID, checksum, fromAddress, srcLvsUUID string, size uint64) (*api.BackingImage, error) {
	input := map[string]string{
		"name":             name,
		"backingImageUUID": backingImageUUID,
		"checksum":         checksum,
		"diskUUID":         diskUUID,
	}

	if err := validateProxyMethodParameters(input); err != nil {
		return nil, errors.Wrap(err, "failed to create backing image")
	}

	if size == 0 {
		return nil, fmt.Errorf("failed to create backing image, size should not be zero")
	}

	client := c.service
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	resp, err := client.SPDKBackingImageCreate(ctx, &rpc.SPDKBackingImageCreateRequest{
		Name:             name,
		BackingImageUuid: backingImageUUID,
		DiskUuid:         diskUUID,
		Size:             size,
		Checksum:         checksum,
		FromAddress:      fromAddress,
		SrcLvsUuid:       srcLvsUUID,
	})
	if err != nil {
		return nil, err
	}

	return api.RPCToBackingImage(resp)
}

func (c *ProxyClient) SPDKBackingImageDelete(name, diskUUID string) error {
	input := map[string]string{
		"name":     name,
		"diskUUID": diskUUID,
	}

	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to delete backing image")
	}

	client := c.service
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	_, err := client.SPDKBackingImageDelete(ctx, &rpc.SPDKBackingImageDeleteRequest{
		Name:     name,
		DiskUuid: diskUUID,
	})
	return err
}

func (c *ProxyClient) SPDKBackingImageGet(name, diskUUID string) (*api.BackingImage, error) {
	input := map[string]string{
		"name":     name,
		"diskUUID": diskUUID,
	}

	if err := validateProxyMethodParameters(input); err != nil {
		return nil, errors.Wrap(err, "failed to get backing image")
	}

	client := c.service
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	resp, err := client.SPDKBackingImageGet(ctx, &rpc.SPDKBackingImageGetRequest{
		Name:     name,
		DiskUuid: diskUUID,
	})
	if err != nil {
		return nil, err
	}
	return api.RPCToBackingImage(resp)
}

func (c *ProxyClient) SPDKBackingImageList() (map[string]*api.BackingImage, error) {
	client := c.service
	ctx, cancel := context.WithTimeout(context.Background(), types.GRPCServiceTimeout)
	defer cancel()

	resp, err := client.SPDKBackingImageList(ctx, &emptypb.Empty{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to list backing images")
	}
	return api.RPCToBackingImageList(resp), nil
}

func (c *ProxyClient) SPDKBackingImageWatch(ctx context.Context) (*api.BackingImageStream, error) {
	client := c.service
	stream, err := client.SPDKBackingImageWatch(ctx, &emptypb.Empty{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to open backing image update stream")
	}

	return api.NewBackingImageStream(stream), nil
}
</file>

<file path="pkg/client/proxy_backup.go">
package client

import (
	"encoding/json"
	"fmt"

	"github.com/cockroachdb/errors"
	"google.golang.org/protobuf/types/known/emptypb"

	rpc "github.com/longhorn/types/pkg/generated/imrpc"
)

func (c *ProxyClient) CleanupBackupMountPoints() (err error) {
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err = c.service.CleanupBackupMountPoints(ctx, &emptypb.Empty{})
	if err != nil {
		return err
	}
	return nil
}

func (c *ProxyClient) SnapshotBackup(dataEngine, engineName, volumeName, serviceAddress, backupName,
	snapshotName, backupTarget, backingImageName, backingImageChecksum, compressionMethod string, concurrentLimit int,
	storageClassName string, labels map[string]string, envs []string, parameters map[string]string) (backupID, replicaAddress string, err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return "", "", errors.Wrap(err, "failed to backup snapshot")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return "", "", fmt.Errorf("failed to backup snapshot: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to backup snapshot %v to %v", c.getProxyErrorPrefix(serviceAddress), snapshotName, backupName)
	}()

	req := &rpc.EngineSnapshotBackupRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		Envs:                 envs,
		BackupName:           backupName,
		SnapshotName:         snapshotName,
		BackupTarget:         backupTarget,
		BackingImageName:     backingImageName,
		BackingImageChecksum: backingImageChecksum,
		CompressionMethod:    compressionMethod,
		ConcurrentLimit:      int32(concurrentLimit),
		StorageClassName:     storageClassName,
		Labels:               labels,
		Parameters:           parameters,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	recv, err := c.service.SnapshotBackup(ctx, req)
	if err != nil {
		return "", "", err
	}
	return recv.BackupId, recv.Replica, nil
}

func (c *ProxyClient) SnapshotBackupStatus(dataEngine, engineName, volumeName, serviceAddress, backupName,
	replicaAddress, replicaName string) (status *SnapshotBackupStatus, err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
		"backupName":     backupName,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return nil, errors.Wrap(err, "failed to get backup status")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return nil, fmt.Errorf("failed to get backup status: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to get %v backup status", c.getProxyErrorPrefix(serviceAddress), backupName)
	}()

	req := &rpc.EngineSnapshotBackupStatusRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		BackupName:     backupName,
		ReplicaAddress: replicaAddress,
		// For now, it is unlikely we actually know replicaName. Pass it anyway, as an empty string will not cause a
		// validation failure and this may change in the future.
		ReplicaName: replicaName,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	recv, err := c.service.SnapshotBackupStatus(ctx, req)
	if err != nil {
		return nil, err
	}

	status = &SnapshotBackupStatus{
		Progress:       int(recv.Progress),
		BackupURL:      recv.BackupUrl,
		Error:          recv.Error,
		SnapshotName:   recv.SnapshotName,
		State:          recv.State,
		ReplicaAddress: recv.ReplicaAddress,
	}
	return status, nil
}

func (c *ProxyClient) BackupRestore(dataEngine, engineName, volumeName, serviceAddress, url, target,
	backupVolumeName string, envs []string, concurrentLimit int) (err error) {
	input := map[string]string{
		"engineName":       engineName,
		"volumeName":       volumeName,
		"serviceAddress":   serviceAddress,
		"url":              url,
		"target":           target,
		"backupVolumeName": backupVolumeName,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to restore backup to volume")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to restore backup to volume: invalid data engine %v", dataEngine)
	}

	defer func() {
		if _, ok := err.(TaskError); ok {
			return
		}

		err = errors.Wrapf(err, "%v failed to restore backup %v to volume %v", c.getProxyErrorPrefix(serviceAddress), url, volumeName)
	}()

	req := &rpc.EngineBackupRestoreRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			// This is the name we will use for validation when communicating with the restoring engine.
			VolumeName: volumeName,
		},
		Envs:   envs,
		Url:    url,
		Target: target,
		// Historically, we have passed backupVolumeName as VolumeName here.
		VolumeName:      backupVolumeName,
		ConcurrentLimit: int32(concurrentLimit),
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	recv, err := c.service.BackupRestore(ctx, req)
	if err != nil {
		return err
	}

	if recv.TaskError != nil {
		var taskErr TaskError
		if jsonErr := json.Unmarshal(recv.TaskError, &taskErr); jsonErr != nil {
			err = errors.Wrapf(jsonErr, "cannot unmarshal the restore error, maybe it's not caused by the replica restore failure: %s", recv.TaskError)
			return err
		}

		err = taskErr
		return err
	}

	return nil
}

func (c *ProxyClient) BackupRestoreStatus(dataEngine, engineName, volumeName, serviceAddress string) (status map[string]*BackupRestoreStatus, err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return nil, errors.Wrap(err, "failed to get backup restore status")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return nil, fmt.Errorf("failed to get backup restore status: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to get backup restore status", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.ProxyEngineRequest{
		Address: serviceAddress,
		// nolint:all replaced with DataEngine
		BackendStoreDriver: rpc.BackendStoreDriver(driver),
		DataEngine:         rpc.DataEngine(driver),
		EngineName:         engineName,
		VolumeName:         volumeName,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	recv, err := c.service.BackupRestoreStatus(ctx, req)
	if err != nil {
		return nil, err
	}

	status = map[string]*BackupRestoreStatus{}
	for k, v := range recv.Status {
		status[k] = &BackupRestoreStatus{
			IsRestoring:            v.IsRestoring,
			LastRestored:           v.LastRestored,
			CurrentRestoringBackup: v.CurrentRestoringBackup,
			Progress:               int(v.Progress),
			Error:                  v.Error,
			Filename:               v.Filename,
			State:                  v.State,
			BackupURL:              v.BackupUrl,
		}
	}
	return status, nil
}
</file>

<file path="pkg/client/proxy_metrics.go">
package client

import (
	"fmt"

	"github.com/cockroachdb/errors"

	rpc "github.com/longhorn/types/pkg/generated/imrpc"
)

func (c *ProxyClient) MetricsGet(dataEngine, engineName, volumeName, serviceAddress string) (metrics *Metrics, err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
		"dataEngine":     dataEngine,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return nil, errors.Wrap(err, "failed to get metrics for volume")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return nil, fmt.Errorf("failed to get metrics for volume: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to get metrics for volume", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.ProxyEngineRequest{
		Address:    serviceAddress,
		EngineName: engineName,
		VolumeName: volumeName,
		DataEngine: rpc.DataEngine(driver),
	}

	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	resp, err := c.service.MetricsGet(ctx, req)
	if err != nil {
		return nil, err
	}

	return &Metrics{
		ReadThroughput:  resp.Metrics.ReadThroughput,
		WriteThroughput: resp.Metrics.WriteThroughput,
		ReadIOPS:        resp.Metrics.ReadIOPS,
		WriteIOPS:       resp.Metrics.WriteIOPS,
		ReadLatency:     resp.Metrics.ReadLatency,
		WriteLatency:    resp.Metrics.WriteLatency,
	}, nil
}
</file>

<file path="pkg/client/proxy_replica.go">
package client

import (
	"fmt"

	"github.com/cockroachdb/errors"

	etypes "github.com/longhorn/longhorn-engine/pkg/types"
	rpc "github.com/longhorn/types/pkg/generated/imrpc"
)

func (c *ProxyClient) ReplicaAdd(dataEngine, engineName, volumeName, serviceAddress, replicaName,
	replicaAddress string, restore bool, size, currentSize int64, fileSyncHTTPClientTimeout int,
	fastSync bool, localSync *etypes.FileLocalSync, grpcTimeoutSeconds int64) (err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
		"replicaName":    replicaName,
		"replicaAddress": replicaAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to add replica for volume")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to add replica for volume: invalid data engine %v", dataEngine)
	}

	defer func() {
		if restore {
			err = errors.Wrapf(err, "%v failed to add restore replica %v for volume", c.getProxyErrorPrefix(serviceAddress), replicaAddress)
		} else {
			err = errors.Wrapf(err, "%v failed to add replica %v for volume", c.getProxyErrorPrefix(serviceAddress), replicaAddress)
		}
	}()

	req := &rpc.EngineReplicaAddRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		ReplicaName:               replicaName,
		ReplicaAddress:            replicaAddress,
		Restore:                   restore,
		Size:                      size,
		CurrentSize:               currentSize,
		FastSync:                  fastSync,
		FileSyncHttpClientTimeout: int32(fileSyncHTTPClientTimeout),
		GrpcTimeoutSeconds:        grpcTimeoutSeconds,
	}

	if localSync != nil {
		req.LocalSync = &rpc.EngineReplicaLocalSync{
			SourcePath: localSync.SourcePath,
			TargetPath: localSync.TargetPath,
		}
	}

	ctx, cancel := getContextWithGRPCLongTimeout(c.ctx, grpcTimeoutSeconds)
	defer cancel()
	_, err = c.service.ReplicaAdd(ctx, req)
	if err != nil {
		return err
	}

	return nil
}

func (c *ProxyClient) ReplicaList(dataEngine, engineName, volumeName,
	serviceAddress string) (rInfoList []*etypes.ControllerReplicaInfo, err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return nil, errors.Wrap(err, "failed to list replicas for volume")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return nil, fmt.Errorf("failed to list replicas for volume: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to list replicas for volume", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.ProxyEngineRequest{
		Address:    serviceAddress,
		EngineName: engineName,
		// nolint:all replaced with DataEngine
		BackendStoreDriver: rpc.BackendStoreDriver(driver),
		DataEngine:         rpc.DataEngine(driver),
		VolumeName:         volumeName,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	resp, err := c.service.ReplicaList(ctx, req)
	if err != nil {
		return nil, err
	}

	for _, cr := range resp.ReplicaList.Replicas {
		rInfoList = append(rInfoList, &etypes.ControllerReplicaInfo{
			Address: cr.Address.Address,
			Mode:    etypes.GRPCReplicaModeToReplicaMode(cr.Mode),
		})
	}

	return rInfoList, nil
}

func (c *ProxyClient) ReplicaRebuildingStatus(dataEngine, engineName, volumeName,
	serviceAddress string) (status map[string]*ReplicaRebuildStatus, err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return nil, errors.Wrap(err, "failed to get replicas rebuilding status")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return nil, fmt.Errorf("failed to get replicas rebuilding status: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to get replicas rebuilding status", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.ProxyEngineRequest{
		Address:    serviceAddress,
		EngineName: engineName,
		// nolint:all replaced with DataEngine
		BackendStoreDriver: rpc.BackendStoreDriver(driver),
		DataEngine:         rpc.DataEngine(driver),
		VolumeName:         volumeName,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	recv, err := c.service.ReplicaRebuildingStatus(ctx, req)
	if err != nil {
		return status, err
	}

	status = make(map[string]*ReplicaRebuildStatus)
	for k, v := range recv.Status {
		status[k] = &ReplicaRebuildStatus{
			Error:                  v.Error,
			IsRebuilding:           v.IsRebuilding,
			Progress:               int(v.Progress),
			State:                  v.State,
			FromReplicaAddressList: v.FromReplicaAddressList,
		}
	}
	return status, nil
}

func (c *ProxyClient) ReplicaRebuildingQosSet(dataEngine, engineName, volumeName,
	serviceAddress string, qosLimitMbps int64) (err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to set replicas rebuilding qos set")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to set replicas rebuilding qos set: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to set replicas rebuilding qos set", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.EngineReplicaRebuildingQosSetRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		QosLimitMbps: qosLimitMbps,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err = c.service.ReplicaRebuildingQosSet(ctx, req)
	return err
}

func (c *ProxyClient) ReplicaVerifyRebuild(dataEngine, engineName, volumeName, serviceAddress,
	replicaAddress, replicaName string) (err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
		"replicaAddress": replicaAddress,
		"replicaName":    replicaName,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to verify replica rebuild")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to verify replica rebuild: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to verify replica %v rebuild", c.getProxyErrorPrefix(serviceAddress), replicaAddress)
	}()

	req := &rpc.EngineReplicaVerifyRebuildRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		ReplicaAddress: replicaAddress,
		ReplicaName:    replicaName,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err = c.service.ReplicaVerifyRebuild(ctx, req)
	if err != nil {
		return err
	}

	return nil
}

func (c *ProxyClient) ReplicaRemove(dataEngine, serviceAddress, engineName, replicaAddress, replicaName string) (err error) {
	input := map[string]string{
		"serviceAddress": serviceAddress,
		"engineName":     engineName,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to remove replica for volume")
	}

	if replicaAddress == "" && replicaName == "" {
		return fmt.Errorf("failed to remove replica for volume: replica address and name are both empty")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to remove replica for volume: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to remove replica %v for volume", c.getProxyErrorPrefix(serviceAddress), replicaAddress)
	}()

	req := &rpc.EngineReplicaRemoveRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
		},
		ReplicaAddress: replicaAddress,
		ReplicaName:    replicaName,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err = c.service.ReplicaRemove(ctx, req)
	if err != nil {
		return err
	}

	return nil
}

func (c *ProxyClient) ReplicaModeUpdate(dataEngine, serviceAddress, replicaAddress string, mode string) (err error) {
	input := map[string]string{
		"serviceAddress": serviceAddress,
		"replicaAddress": replicaAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to remove replica for volume")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to remove replica for volume: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to update replica %v mode for volume", c.getProxyErrorPrefix(serviceAddress), replicaAddress)
	}()

	req := &rpc.EngineReplicaModeUpdateRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address: serviceAddress,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
		},
		ReplicaAddress: replicaAddress,
		Mode:           etypes.ReplicaModeToGRPCReplicaMode(etypes.Mode(mode)),
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err = c.service.ReplicaModeUpdate(ctx, req)
	if err != nil {
		return err
	}

	return nil
}

func (c *ProxyClient) ReplicaRebuildConcurrentSyncLimitSet(dataEngine, engineName, volumeName, serviceAddress string,
	limit int) (err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to set replica rebuilding concurrent sync limit")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to set replica rebuilding concurrent sync limit: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to set replica rebuilding concurrent sync limit", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.EngineReplicaRebuildConcurrentSyncLimitSetRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			DataEngine: rpc.DataEngine(driver),
			VolumeName: volumeName,
		},
		Limit: int32(limit),
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	if _, err = c.service.ReplicaRebuildConcurrentSyncLimitSet(ctx, req); err != nil {
		return err
	}

	return nil
}

func (c *ProxyClient) ReplicaRebuildConcurrentSyncLimitGet(dataEngine, engineName, volumeName,
	serviceAddress string) (limit int, err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return 0, errors.Wrap(err, "failed to get replica rebuilding concurrent sync limit")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return 0, fmt.Errorf("failed to get replica rebuilding concurrent sync limit: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to get replica rebuilding concurrent sync limit", c.getProxyErrorPrefix(serviceAddress))
	}()
	req := &rpc.ProxyEngineRequest{
		Address:    serviceAddress,
		EngineName: engineName,
		DataEngine: rpc.DataEngine(driver),
		VolumeName: volumeName,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	resp, err := c.service.ReplicaRebuildConcurrentSyncLimitGet(ctx, req)
	if err != nil {
		return 0, err
	}

	limit = int(resp.Limit)
	return limit, nil
}
</file>

<file path="pkg/client/proxy_snapshot.go">
package client

import (
	"fmt"

	"github.com/cockroachdb/errors"

	"github.com/longhorn/types/pkg/generated/enginerpc"

	etypes "github.com/longhorn/longhorn-engine/pkg/types"
	eutil "github.com/longhorn/longhorn-engine/pkg/util"
	rpc "github.com/longhorn/types/pkg/generated/imrpc"
)

func (c *ProxyClient) VolumeSnapshot(dataEngine, engineName, volumeName, serviceAddress,
	volumeSnapshotName string, labels map[string]string, freezeFilesystem bool) (snapshotName string, err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return "", errors.Wrap(err, "failed to snapshot volume")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return "", fmt.Errorf("failed to snapshot volume: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to snapshot volume", c.getProxyErrorPrefix(serviceAddress))
	}()

	for key, value := range labels {
		if errList := eutil.IsQualifiedName(key); len(errList) > 0 {
			err = errors.Errorf("invalid key %v for label: %v", key, errList[0])
			return "", err
		}

		// We don't need to validate the Label value since we're allowing for any form of data to be stored, similar
		// to Kubernetes Annotations. Of course, we should make sure it isn't empty.
		if value == "" {
			err = errors.Errorf("invalid empty value for label with key %v", key)
			return "", err
		}
	}

	req := &rpc.EngineVolumeSnapshotRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		SnapshotVolume: &enginerpc.VolumeSnapshotRequest{
			Name:             volumeSnapshotName,
			Labels:           labels,
			FreezeFilesystem: freezeFilesystem,
		},
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	recv, err := c.service.VolumeSnapshot(ctx, req)
	if err != nil {
		return "", err
	}
	return recv.Snapshot.Name, nil
}

func (c *ProxyClient) SnapshotList(dataEngine, engineName, volumeName,
	serviceAddress string) (snapshotDiskInfo map[string]*etypes.DiskInfo, err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return nil, errors.Wrap(err, "failed to list snapshots")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return nil, fmt.Errorf("failed to list snapshots: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to list snapshots", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.ProxyEngineRequest{
		Address:    serviceAddress,
		EngineName: engineName,
		// nolint:all replaced with DataEngine
		BackendStoreDriver: rpc.BackendStoreDriver(driver),
		DataEngine:         rpc.DataEngine(driver),
		VolumeName:         volumeName,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	resp, err := c.service.SnapshotList(ctx, req)
	if err != nil {
		return nil, err
	}

	snapshotDiskInfo = map[string]*etypes.DiskInfo{}
	for k, v := range resp.Disks {
		if v.Children == nil {
			v.Children = map[string]bool{}
		}
		if v.Labels == nil {
			v.Labels = map[string]string{}
		}
		snapshotDiskInfo[k] = &etypes.DiskInfo{
			Name:        v.Name,
			Parent:      v.Parent,
			Children:    v.Children,
			Removed:     v.Removed,
			UserCreated: v.UserCreated,
			Created:     v.Created,
			Size:        v.Size,
			Labels:      v.Labels,
		}
	}
	return snapshotDiskInfo, nil
}

func (c *ProxyClient) SnapshotClone(dataEngine, engineName, volumeName, serviceAddress,
	snapshotName, fromEngineAddress, fromVolumeName, fromEngineName string, fileSyncHTTPClientTimeout int,
	grpcTimeoutSeconds int64, cloneMode string) (err error) {
	input := map[string]string{
		"engineName":        engineName,
		"volumeName":        volumeName,
		"serviceAddress":    serviceAddress,
		"snapshotName":      snapshotName,
		"fromEngineAddress": fromEngineAddress,
		"fromVolumeName":    fromVolumeName,
		"fromEngineName":    fromEngineName,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to clone snapshot")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to clone snapshot: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to clone snapshot %v from %v", c.getProxyErrorPrefix(serviceAddress),
			snapshotName, fromEngineAddress)
	}()

	req := &rpc.EngineSnapshotCloneRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		FromEngineAddress:         fromEngineAddress,
		SnapshotName:              snapshotName,
		ExportBackingImageIfExist: false,
		FileSyncHttpClientTimeout: int32(fileSyncHTTPClientTimeout),
		FromEngineName:            fromEngineName,
		FromVolumeName:            fromVolumeName,
		GrpcTimeoutSeconds:        grpcTimeoutSeconds,
		CloneMode:                 getCloneMode(cloneMode),
	}
	ctx, cancel := getContextWithGRPCLongTimeout(c.ctx, grpcTimeoutSeconds)
	defer cancel()
	_, err = c.service.SnapshotClone(ctx, req)
	if err != nil {
		return err
	}

	return nil
}

func (c *ProxyClient) SnapshotCloneStatus(dataEngine, engineName, volumeName, serviceAddress string) (status map[string]*SnapshotCloneStatus, err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return nil, errors.Wrap(err, "failed to get snapshot clone status")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return nil, fmt.Errorf("failed to get snapshot clone status: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to get snapshot clone status", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.ProxyEngineRequest{
		Address:    serviceAddress,
		EngineName: engineName,
		// nolint:all replaced with DataEngine
		BackendStoreDriver: rpc.BackendStoreDriver(driver),
		DataEngine:         rpc.DataEngine(driver),
		VolumeName:         volumeName,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	recv, err := c.service.SnapshotCloneStatus(ctx, req)
	if err != nil {
		return nil, err
	}

	status = map[string]*SnapshotCloneStatus{}
	for k, v := range recv.Status {
		status[k] = &SnapshotCloneStatus{
			IsCloning:          v.IsCloning,
			Error:              v.Error,
			Progress:           int(v.Progress),
			State:              v.State,
			FromReplicaAddress: v.FromReplicaAddress,
			SnapshotName:       v.SnapshotName,
		}
	}
	return status, nil
}

func (c *ProxyClient) SnapshotRevert(dataEngine, engineName, volumeName, serviceAddress string,
	name string) (err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
		"name":           name,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to revert volume to snapshot")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to revert volume to snapshot: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to revert volume to snapshot %v", c.getProxyErrorPrefix(serviceAddress), name)
	}()

	if name == etypes.VolumeHeadName {
		err = errors.Errorf("invalid operation: cannot revert to %v", etypes.VolumeHeadName)
		return err
	}

	req := &rpc.EngineSnapshotRevertRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		Name: name,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err = c.service.SnapshotRevert(ctx, req)
	if err != nil {
		return err
	}

	return nil
}

func (c *ProxyClient) SnapshotPurge(dataEngine, engineName, volumeName, serviceAddress string,
	skipIfInProgress bool) (err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to purge snapshots")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to purge snapshots: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to purge snapshots", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.EngineSnapshotPurgeRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		SkipIfInProgress: skipIfInProgress,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err = c.service.SnapshotPurge(ctx, req)
	if err != nil {
		return err
	}

	return nil
}

func (c *ProxyClient) SnapshotPurgeStatus(dataEngine, engineName, volumeName, serviceAddress string) (status map[string]*SnapshotPurgeStatus, err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return nil, errors.Wrap(err, "failed to get snapshot purge status")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return nil, fmt.Errorf("failed to get snapshot purge status: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to get snapshot purge status", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.ProxyEngineRequest{
		Address:    serviceAddress,
		EngineName: engineName,
		// nolint:all replaced with DataEngine
		BackendStoreDriver: rpc.BackendStoreDriver(driver),
		DataEngine:         rpc.DataEngine(driver),
		VolumeName:         volumeName,
	}

	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	recv, err := c.service.SnapshotPurgeStatus(ctx, req)
	if err != nil {
		return nil, err
	}

	status = make(map[string]*SnapshotPurgeStatus)
	for k, v := range recv.Status {
		status[k] = &SnapshotPurgeStatus{
			Error:     v.Error,
			IsPurging: v.IsPurging,
			Progress:  int(v.Progress),
			State:     v.State,
		}
	}
	return status, nil
}

func (c *ProxyClient) SnapshotRemove(dataEngine, engineName, volumeName, serviceAddress string,
	names []string) (err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrapf(err, "failed to remove snapshot %v", names)
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to remove snapshot: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to remove snapshot %v", c.getProxyErrorPrefix(serviceAddress), names)
	}()

	req := &rpc.EngineSnapshotRemoveRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		Names: names,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err = c.service.SnapshotRemove(ctx, req)
	if err != nil {
		return err
	}

	return nil
}

func (c *ProxyClient) SnapshotHash(dataEngine, engineName, volumeName, serviceAddress string,
	snapshotName string, rehash bool) (err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to hash snapshot")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to hash snapshot: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to hash snapshot", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.EngineSnapshotHashRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		SnapshotName: snapshotName,
		Rehash:       rehash,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err = c.service.SnapshotHash(ctx, req)
	if err != nil {
		return err
	}
	return nil
}

func (c *ProxyClient) SnapshotHashStatus(dataEngine, engineName, volumeName, serviceAddress,
	snapshotName string) (status map[string]*SnapshotHashStatus, err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return nil, errors.Wrap(err, "failed to get snapshot hash status")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return nil, fmt.Errorf("failed to get snapshot hash status: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to get snapshot hash status", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.EngineSnapshotHashStatusRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		SnapshotName: snapshotName,
	}

	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	recv, err := c.service.SnapshotHashStatus(ctx, req)
	if err != nil {
		return nil, err
	}

	status = make(map[string]*SnapshotHashStatus)
	for k, v := range recv.Status {
		status[k] = &SnapshotHashStatus{
			State:             v.State,
			Checksum:          v.Checksum,
			Error:             v.Error,
			SilentlyCorrupted: v.SilentlyCorrupted,
		}
	}

	return status, nil
}
</file>

<file path="pkg/client/proxy_types.go">
package client

type SnapshotCloneStatus struct {
	IsCloning          bool
	Error              string
	Progress           int
	State              string
	FromReplicaAddress string
	SnapshotName       string
}

type SnapshotPurgeStatus struct {
	Error     string
	IsPurging bool
	Progress  int
	State     string
}

type SnapshotBackupStatus struct {
	Progress       int
	BackupURL      string
	Error          string
	SnapshotName   string
	State          string
	ReplicaAddress string
}

type BackupRestoreStatus struct {
	IsRestoring            bool
	LastRestored           string
	CurrentRestoringBackup string
	Progress               int
	Error                  string
	Filename               string
	State                  string
	BackupURL              string
}

type EngineBackupVolumeInfo struct {
	Name                 string
	Size                 int64
	Labels               map[string]string
	Created              string
	LastBackupName       string
	LastBackupAt         string
	DataStored           int64
	Messages             map[string]string
	Backups              map[string]*EngineBackupInfo
	BackingImageName     string
	BackingImageChecksum string
}

type EngineBackupInfo struct {
	Name                   string
	URL                    string
	SnapshotName           string
	SnapshotCreated        string
	Created                string
	Size                   int64
	Labels                 map[string]string
	IsIncremental          bool
	VolumeName             string
	VolumeSize             int64
	VolumeCreated          string
	VolumeBackingImageName string
	Messages               map[string]string
}

type ReplicaRebuildStatus struct {
	Error        string
	IsRebuilding bool
	Progress     int
	State        string
	// Deprecated: use FromReplicaAddressList instead
	FromReplicaAddress     string
	FromReplicaAddressList []string
	AppliedRebuildingMBps  int64
}

type SnapshotHashStatus struct {
	State             string
	Checksum          string
	Error             string
	SilentlyCorrupted bool
}

type Metrics struct {
	ReadThroughput  uint64
	WriteThroughput uint64
	ReadLatency     uint64
	WriteLatency    uint64
	ReadIOPS        uint64
	WriteIOPS       uint64
}
</file>

<file path="pkg/client/proxy_volume.go">
package client

import (
	"fmt"
	"strconv"

	"github.com/cockroachdb/errors"

	etypes "github.com/longhorn/longhorn-engine/pkg/types"
	"github.com/longhorn/types/pkg/generated/enginerpc"
	rpc "github.com/longhorn/types/pkg/generated/imrpc"
)

func (c *ProxyClient) VolumeGet(dataEngine, engineName, volumeName, serviceAddress string) (info *etypes.VolumeInfo, err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return nil, errors.Wrap(err, "failed to get volume")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return nil, fmt.Errorf("failed to get volume: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to get volume", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.ProxyEngineRequest{
		Address:    serviceAddress,
		EngineName: engineName,
		// nolint:all replaced with DataEngine
		BackendStoreDriver: rpc.BackendStoreDriver(driver),
		DataEngine:         rpc.DataEngine(driver),
		VolumeName:         volumeName,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	resp, err := c.service.VolumeGet(ctx, req)
	if err != nil {
		return nil, err
	}

	info = &etypes.VolumeInfo{
		Name:                      resp.Volume.Name,
		Size:                      resp.Volume.Size,
		ReplicaCount:              int(resp.Volume.ReplicaCount),
		Endpoint:                  resp.Volume.Endpoint,
		Frontend:                  resp.Volume.Frontend,
		FrontendState:             resp.Volume.FrontendState,
		IsExpanding:               resp.Volume.IsExpanding,
		LastExpansionError:        resp.Volume.LastExpansionError,
		LastExpansionFailedAt:     resp.Volume.LastExpansionFailedAt,
		UnmapMarkSnapChainRemoved: resp.Volume.UnmapMarkSnapChainRemoved,
		SnapshotMaxCount:          int(resp.Volume.SnapshotMaxCount),
		SnapshotMaxSize:           resp.Volume.SnapshotMaxSize,
	}
	return info, nil
}

func (c *ProxyClient) VolumeExpand(dataEngine, engineName, volumeName, serviceAddress string,
	size int64) (err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to expand volume")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to expand volume: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to expand volume", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.EngineVolumeExpandRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		Expand: &enginerpc.VolumeExpandRequest{
			Size: size,
		},
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err = c.service.VolumeExpand(ctx, req)
	if err != nil {
		return err
	}

	return nil
}

func (c *ProxyClient) VolumeFrontendStart(dataEngine, engineName, volumeName, serviceAddress, frontendName string) (err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
		"frontendName":   frontendName,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to start volume frontend")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to start volume frontend: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to start volume frontend %v", c.getProxyErrorPrefix(serviceAddress), frontendName)
	}()

	req := &rpc.EngineVolumeFrontendStartRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		FrontendStart: &enginerpc.VolumeFrontendStartRequest{
			Frontend: frontendName,
		},
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err = c.service.VolumeFrontendStart(ctx, req)
	if err != nil {
		return err
	}

	return nil
}

func (c *ProxyClient) VolumeFrontendShutdown(dataEngine, engineName, volumeName, serviceAddress string) (err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to shutdown volume frontend")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to shutdown volume frontend: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to shutdown volume frontend", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.ProxyEngineRequest{
		Address:    serviceAddress,
		EngineName: engineName,
		// nolint:all replaced with DataEngine
		BackendStoreDriver: rpc.BackendStoreDriver(driver),
		DataEngine:         rpc.DataEngine(driver),
		VolumeName:         volumeName,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err = c.service.VolumeFrontendShutdown(ctx, req)
	if err != nil {
		return err
	}

	return nil
}

func (c *ProxyClient) VolumeUnmapMarkSnapChainRemovedSet(dataEngine, engineName, volumeName, serviceAddress string, enabled bool) (err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
		"enabled":        strconv.FormatBool(enabled),
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to set volume flag UnmapMarkSnapChainRemoved")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to set volume flag UnmapMarkSnapChainRemoved: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to set UnmapMarkSnapChainRemoved", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.EngineVolumeUnmapMarkSnapChainRemovedSetRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		UnmapMarkSnap: &enginerpc.VolumeUnmapMarkSnapChainRemovedSetRequest{Enabled: enabled},
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err = c.service.VolumeUnmapMarkSnapChainRemovedSet(ctx, req)
	if err != nil {
		return err
	}

	return nil
}

func (c *ProxyClient) VolumeSnapshotMaxCountSet(dataEngine, engineName, volumeName,
	serviceAddress string, count int) (err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
		"count":          strconv.Itoa(count),
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to set volume flag SnapshotMaxCount")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to set volume flag SnapshotMaxCount: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to set SnapshotMaxCount", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.EngineVolumeSnapshotMaxCountSetRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		Count: &enginerpc.VolumeSnapshotMaxCountSetRequest{Count: int32(count)},
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err = c.service.VolumeSnapshotMaxCountSet(ctx, req)
	if err != nil {
		return err
	}

	return nil
}

func (c *ProxyClient) VolumeSnapshotMaxSizeSet(dataEngine, engineName, volumeName,
	serviceAddress string, size int64) (err error) {
	input := map[string]string{
		"engineName":     engineName,
		"volumeName":     volumeName,
		"serviceAddress": serviceAddress,
		"size":           strconv.FormatInt(size, 10),
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return errors.Wrap(err, "failed to set volume flag SnapshotMaxSize")
	}

	driver, ok := rpc.DataEngine_value[getDataEngine(dataEngine)]
	if !ok {
		return fmt.Errorf("failed to set volume flag SnapshotMaxSize: invalid data engine %v", dataEngine)
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to set SnapshotMaxSize", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.EngineVolumeSnapshotMaxSizeSetRequest{
		ProxyEngineRequest: &rpc.ProxyEngineRequest{
			Address:    serviceAddress,
			EngineName: engineName,
			// nolint:all replaced with DataEngine
			BackendStoreDriver: rpc.BackendStoreDriver(driver),
			DataEngine:         rpc.DataEngine(driver),
			VolumeName:         volumeName,
		},
		Size: &enginerpc.VolumeSnapshotMaxSizeSetRequest{Size: size},
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err = c.service.VolumeSnapshotMaxSizeSet(ctx, req)
	if err != nil {
		return err
	}

	return nil
}

func (c *ProxyClient) RemountReadOnlyVolume(volumeName string) (err error) {
	if volumeName == "" {
		return fmt.Errorf("failed to remount volume, volume name is empty")
	}

	req := &rpc.RemountVolumeRequest{
		VolumeName: volumeName,
	}

	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err = c.service.RemountReadOnlyVolume(ctx, req)
	if err != nil {
		return err
	}

	return nil
}
</file>

<file path="pkg/client/proxy.go">
package client

import (
	"context"
	"crypto/tls"
	"fmt"
	"time"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"google.golang.org/grpc"
	"google.golang.org/grpc/connectivity"

	healthpb "google.golang.org/grpc/health/grpc_health_v1"

	emeta "github.com/longhorn/longhorn-engine/pkg/meta"
	eclient "github.com/longhorn/longhorn-engine/pkg/replica/client"
	rpc "github.com/longhorn/types/pkg/generated/imrpc"

	"github.com/longhorn/longhorn-instance-manager/pkg/meta"
	"github.com/longhorn/longhorn-instance-manager/pkg/util"
)

var (
	ErrParameterFmt = "missing required %v parameter"
)

func validateProxyMethodParameters(input map[string]string) error {
	for k, v := range input {
		if v == "" {
			return errors.Errorf(ErrParameterFmt, k)
		}
	}
	return nil
}

type ServiceContext struct {
	cc *grpc.ClientConn

	ctx  context.Context
	quit context.CancelFunc

	service rpc.ProxyEngineServiceClient
	health  healthpb.HealthClient
}

func (s ServiceContext) GetConnectionState() connectivity.State {
	return s.cc.GetState()
}

func (c *ProxyClient) Close() error {
	c.quit()
	if err := c.cc.Close(); err != nil {
		return errors.Wrap(err, "failed to close proxy gRPC connection")
	}
	return nil
}

type ProxyClient struct {
	ServiceURL string
	ServiceContext

	Version int
}

func NewProxyClient(ctx context.Context, ctxCancel context.CancelFunc, address string, port int, tlsConfig *tls.Config) (*ProxyClient, error) {
	getServiceCtx := func(serviceUrl string) (ServiceContext, error) {
		connection, err := util.Connect(serviceUrl, tlsConfig)
		if err != nil {
			return ServiceContext{}, errors.Wrapf(err, "cannot connect to ProxyService %v", serviceUrl)
		}
		return ServiceContext{
			cc:      connection,
			ctx:     ctx,
			quit:    ctxCancel,
			service: rpc.NewProxyEngineServiceClient(connection),
			health:  healthpb.NewHealthClient(connection),
		}, nil
	}

	serviceURL := util.GetURL(address, port)
	serviceCtx, err := getServiceCtx(serviceURL)
	if err != nil {
		return nil, err
	}
	logrus.Tracef("Connected to proxy service on %v", serviceURL)

	return &ProxyClient{
		ServiceURL:     serviceURL,
		ServiceContext: serviceCtx,
		Version:        meta.InstanceManagerProxyAPIVersion,
	}, nil
}

func NewProxyClientWithTLS(ctx context.Context, ctxCancel context.CancelFunc, address string, port int, caFile, certFile, keyFile, peerName string) (*ProxyClient, error) {
	tlsConfig, err := util.LoadClientTLS(caFile, certFile, keyFile, peerName)
	if err != nil {
		return nil, errors.Wrap(err, "failed to load tls key pair from file")
	}

	return NewProxyClient(ctx, ctxCancel, address, port, tlsConfig)
}

const (
	// We want to have a slightly bigger timeout on the proxy client-side compared to the actual timeout in the engine/replica because the proxy server adds some delay to the flow
	GRPCServiceTimeout     = eclient.GRPCServiceCommonTimeout * 2
	GRPCServiceLongTimeout = eclient.GRPCServiceLongTimeout + GRPCServiceTimeout
)

func getContextWithGRPCTimeout(parent context.Context) (context.Context, context.CancelFunc) {
	ctx, cancel := context.WithTimeout(parent, GRPCServiceTimeout)
	return ctx, cancel
}

// getContextWithGRPCLongTimeout returns a context with given grpcTimeoutSeconds + GRPCServiceTimeout timeout.
// If grpcTimeoutSeconds is 0, use the default GRPCServiceLongTimeout instead.
func getContextWithGRPCLongTimeout(parent context.Context, grpcTimeoutSeconds int64) (context.Context, context.CancelFunc) {
	grpcTimeout := GRPCServiceLongTimeout
	if grpcTimeoutSeconds > 0 {
		// We want to have a slightly bigger timeout on the proxy client-side compared to the actual timeout in the engine/replica because the proxy server adds some delay to the flow
		grpcTimeout = (time.Second * time.Duration(grpcTimeoutSeconds)) + GRPCServiceTimeout
	}
	return context.WithTimeout(parent, grpcTimeout)
}

func (c *ProxyClient) getProxyErrorPrefix(destination string) string {
	return fmt.Sprintf("proxyServer=%v destination=%v:", c.ServiceURL, destination)
}

func (c *ProxyClient) ServerVersionGet(serviceAddress string) (version *emeta.VersionOutput, err error) {
	input := map[string]string{
		"serviceAddress": serviceAddress,
	}
	if err := validateProxyMethodParameters(input); err != nil {
		return nil, errors.Wrap(err, "failed to get server version")
	}

	defer func() {
		err = errors.Wrapf(err, "%v failed to get server version", c.getProxyErrorPrefix(serviceAddress))
	}()

	req := &rpc.ProxyEngineRequest{
		Address: serviceAddress,
	}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	resp, err := c.service.ServerVersionGet(ctx, req)
	if err != nil {
		return nil, err
	}

	serverVersion := resp.Version
	version = &emeta.VersionOutput{
		Version:                 serverVersion.Version,
		GitCommit:               serverVersion.GitCommit,
		BuildDate:               serverVersion.BuildDate,
		CLIAPIVersion:           int(serverVersion.CliAPIVersion),
		CLIAPIMinVersion:        int(serverVersion.CliAPIMinVersion),
		ControllerAPIVersion:    int(serverVersion.ControllerAPIVersion),
		ControllerAPIMinVersion: int(serverVersion.ControllerAPIMinVersion),
		DataFormatVersion:       int(serverVersion.DataFormatVersion),
		DataFormatMinVersion:    int(serverVersion.DataFormatMinVersion),
	}
	return version, nil
}

func (c *ProxyClient) ClientVersionGet() (version emeta.VersionOutput) {
	logrus.Trace("Getting client version")
	return emeta.GetVersion()
}

func (c *ProxyClient) CheckConnection() error {
	req := &healthpb.HealthCheckRequest{}
	ctx, cancel := getContextWithGRPCTimeout(c.ctx)
	defer cancel()
	_, err := c.health.Check(ctx, req)
	return err
}
</file>

<file path="pkg/client/types.go">
package client

import (
	"fmt"
	"strings"

	rpc "github.com/longhorn/types/pkg/generated/imrpc"
)

const (
	dataEngineV1 = "v1"
	dataEngineV2 = "v2"

	cloneModeFullCopy    = "full-copy"
	cloneModeLinkedClone = "linked-clone"
)

type TaskError struct {
	ReplicaErrors []ReplicaError
}

type ReplicaError struct {
	Address string
	Message string
}

func (e TaskError) Error() string {
	var errs []string
	for _, re := range e.ReplicaErrors {
		errs = append(errs, re.Error())
	}

	if errs == nil {
		return "Unknown"
	}

	return strings.Join(errs, "; ")
}

func (e ReplicaError) Error() string {
	return fmt.Sprintf("%v: %v", e.Address, e.Message)
}

func getDataEngine(dataEngine string) string {
	if strings.HasSuffix(strings.ToLower(dataEngine), dataEngineV2) {
		return rpc.DataEngine_name[int32(rpc.DataEngine_DATA_ENGINE_V2)]
	}

	return rpc.DataEngine_name[int32(rpc.DataEngine_DATA_ENGINE_V1)]
}

func getCloneMode(cloneMode string) rpc.CloneMode {
	if cloneMode == cloneModeLinkedClone {
		return rpc.CloneMode_CLONE_MODE_LINKED_CLONE
	}
	return rpc.CloneMode_CLONE_MODE_FULL_COPY
}
</file>

<file path="pkg/disk/disk.go">
package disk

import (
	"context"
	"fmt"
	"sync"
	"time"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"google.golang.org/protobuf/types/known/emptypb"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/longhorn-spdk-engine/pkg/api"

	spdkclient "github.com/longhorn/longhorn-spdk-engine/pkg/client"
	enginerpc "github.com/longhorn/types/pkg/generated/enginerpc"
	rpc "github.com/longhorn/types/pkg/generated/imrpc"
	spdkrpc "github.com/longhorn/types/pkg/generated/spdkrpc"

	"github.com/longhorn/longhorn-instance-manager/pkg/meta"
	"github.com/longhorn/longhorn-instance-manager/pkg/types"
	"github.com/longhorn/longhorn-instance-manager/pkg/util"
)

const (
	spdkTgtReadinessProbeTimeout = 60 * time.Second
)

type DiskOps interface {
	DiskCreate(context.Context, *rpc.DiskCreateRequest) (*rpc.Disk, error)
	DiskDelete(*rpc.DiskDeleteRequest) (*emptypb.Empty, error)
	DiskGet(req *rpc.DiskGetRequest) (*rpc.Disk, error)
	DiskHealthGet(req *rpc.DiskHealthGetRequest) (*rpc.DiskHealthGetResponse, error)
	DiskReplicaInstanceList(*rpc.DiskReplicaInstanceListRequest) (*rpc.DiskReplicaInstanceListResponse, error)
	DiskReplicaInstanceDelete(*rpc.DiskReplicaInstanceDeleteRequest) (*emptypb.Empty, error)
	MetricsGet(*rpc.DiskGetRequest) (*rpc.DiskMetricsGetReply, error)
}

type FilesystemDiskOps struct{}
type BlockDiskOps struct {
	spdkClient *spdkclient.SPDKClient
}

type Server struct {
	rpc.UnimplementedDiskServiceServer
	sync.RWMutex

	ctx           context.Context
	HealthChecker HealthChecker

	spdkServiceAddress string
	ops                map[rpc.DiskType]DiskOps
}

func NewServer(ctx context.Context, spdkEnabled bool, spdkServiceAddress string) (srv *Server, err error) {
	var spdkClient *spdkclient.SPDKClient

	if spdkEnabled {
		logrus.Info("Disk Server: Creating SPDK client since SPDK is enabled")

		if !util.IsSPDKTgtReady(spdkTgtReadinessProbeTimeout) {
			return nil, fmt.Errorf("spdk_tgt is not ready in %v", spdkTgtReadinessProbeTimeout)
		}

		spdkClient, err = spdkclient.NewSPDKClient(spdkServiceAddress)
		if err != nil {
			return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create SPDK client").Error())
		}
	}

	ops := map[rpc.DiskType]DiskOps{
		rpc.DiskType_filesystem: FilesystemDiskOps{},
		rpc.DiskType_block: BlockDiskOps{
			spdkClient: spdkClient,
		},
	}

	s := &Server{
		ctx:                ctx,
		spdkServiceAddress: spdkServiceAddress,
		HealthChecker:      &GRPCHealthChecker{},
		ops:                ops,
	}

	go s.startMonitoring()

	return s, nil
}

func (s *Server) startMonitoring() {
	<-s.ctx.Done()
	logrus.Infof("%s: stopped monitoring due to the context done", types.DiskGrpcService)
}

func (s *Server) VersionGet(ctx context.Context, req *emptypb.Empty) (*rpc.DiskVersionResponse, error) {
	v := meta.GetDiskServiceVersion()
	return &rpc.DiskVersionResponse{
		Version:   v.Version,
		GitCommit: v.GitCommit,
		BuildDate: v.BuildDate,

		InstanceManagerDiskServiceAPIVersion:    int64(v.InstanceManagerDiskServiceAPIVersion),
		InstanceManagerDiskServiceAPIMinVersion: int64(v.InstanceManagerDiskServiceAPIMinVersion),
	}, nil
}

func (s *Server) DiskCreate(ctx context.Context, req *rpc.DiskCreateRequest) (*rpc.Disk, error) {
	log := logrus.WithFields(logrus.Fields{
		"diskType":   req.DiskType,
		"diskName":   req.DiskName,
		"diskPath":   req.DiskPath,
		"blockSize":  req.BlockSize,
		"diskDriver": req.DiskDriver,
	})

	log.Info("Disk Server: Creating disk")

	if req.DiskName == "" || req.DiskPath == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "disk name and disk path are required")
	}

	ops, ok := s.ops[req.DiskType]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported disk type %v", req.DiskType)
	}
	return ops.DiskCreate(ctx, req)
}

func (ops FilesystemDiskOps) DiskCreate(ctx context.Context, req *rpc.DiskCreateRequest) (*rpc.Disk, error) {
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported disk type %v", req.DiskType)
}

func (ops BlockDiskOps) DiskCreate(ctx context.Context, req *rpc.DiskCreateRequest) (*rpc.Disk, error) {
	ret, err := ops.spdkClient.DiskCreate(req.DiskName, req.DiskUuid, req.DiskPath, req.DiskDriver, req.BlockSize)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, err.Error())
	}
	return spdkDiskToDisk(ret), nil
}

func (s *Server) DiskDelete(ctx context.Context, req *rpc.DiskDeleteRequest) (*emptypb.Empty, error) {
	log := logrus.WithFields(logrus.Fields{
		"diskType":   req.DiskType,
		"diskName":   req.DiskName,
		"diskUUID":   req.DiskUuid,
		"diskPath":   req.DiskPath,
		"diskDriver": req.DiskDriver,
	})

	log.Info("Disk Server: Deleting disk")

	if req.DiskName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "disk name is required")
	}

	ops, ok := s.ops[req.DiskType]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported disk type %v", req.DiskType)
	}
	return ops.DiskDelete(req)
}

func (ops FilesystemDiskOps) DiskDelete(req *rpc.DiskDeleteRequest) (*emptypb.Empty, error) {
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported disk type %v", req.DiskType)
}

func (ops BlockDiskOps) DiskDelete(req *rpc.DiskDeleteRequest) (*emptypb.Empty, error) {
	return &emptypb.Empty{}, ops.spdkClient.DiskDelete(req.DiskName, req.DiskUuid, req.DiskPath, req.DiskDriver)
}

func (s *Server) DiskGet(ctx context.Context, req *rpc.DiskGetRequest) (*rpc.Disk, error) {
	log := logrus.WithFields(logrus.Fields{
		"diskType": req.DiskType,
		"diskName": req.DiskName,
		"diskPath": req.DiskPath,
	})

	log.Trace("Disk Server: Getting disk info")

	if req.DiskName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "disk name is required")
	}

	ops, ok := s.ops[req.DiskType]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported disk type %v", req.DiskType)
	}
	return ops.DiskGet(req)
}

func (ops FilesystemDiskOps) DiskGet(req *rpc.DiskGetRequest) (*rpc.Disk, error) {
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported disk type %v", req.DiskType)
}

func (ops BlockDiskOps) DiskGet(req *rpc.DiskGetRequest) (*rpc.Disk, error) {
	ret, err := ops.spdkClient.DiskGet(req.DiskName, req.DiskPath, req.DiskDriver)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, err.Error())
	}
	return spdkDiskToDisk(ret), nil
}

func (s *Server) DiskHealthGet(ctx context.Context, req *rpc.DiskHealthGetRequest) (*rpc.DiskHealthGetResponse, error) {
	if req.DiskName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "disk name is required")
	}

	ops, ok := s.ops[req.DiskType]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported disk type %v", req.DiskType)
	}
	return ops.DiskHealthGet(req)
}

func (op FilesystemDiskOps) DiskHealthGet(req *rpc.DiskHealthGetRequest) (*rpc.DiskHealthGetResponse, error) {
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported disk type %v", req.DiskType)
}

func (op BlockDiskOps) DiskHealthGet(req *rpc.DiskHealthGetRequest) (*rpc.DiskHealthGetResponse, error) {
	health, err := op.spdkClient.DiskHealthGet(req.DiskName, req.DiskPath, req.DiskDriver)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, err.Error())
	}

	return &rpc.DiskHealthGetResponse{
		ModelNumber:                             health.ModelNumber,
		SerialNumber:                            health.SerialNumber,
		FirmwareRevision:                        health.FirmwareRevision,
		Traddr:                                  health.Traddr,
		CriticalWarning:                         health.CriticalWarning,
		TemperatureCelsius:                      health.TemperatureCelsius,
		AvailableSparePercentage:                health.AvailableSparePercentage,
		AvailableSpareThresholdPercentage:       health.AvailableSpareThresholdPercentage,
		PercentageUsed:                          health.PercentageUsed,
		DataUnitsRead:                           health.DataUnitsRead,
		DataUnitsWritten:                        health.DataUnitsWritten,
		HostReadCommands:                        health.HostReadCommands,
		HostWriteCommands:                       health.HostWriteCommands,
		ControllerBusyTime:                      health.ControllerBusyTime,
		PowerCycles:                             health.PowerCycles,
		PowerOnHours:                            health.PowerOnHours,
		UnsafeShutdowns:                         health.UnsafeShutdowns,
		MediaErrors:                             health.MediaErrors,
		NumErrLogEntries:                        health.NumErrLogEntries,
		WarningTemperatureTimeMinutes:           health.WarningTemperatureTimeMinutes,
		CriticalCompositeTemperatureTimeMinutes: health.CriticalCompositeTemperatureTimeMinutes,
	}, nil
}

func (s *Server) DiskReplicaInstanceList(ctx context.Context, req *rpc.DiskReplicaInstanceListRequest) (*rpc.DiskReplicaInstanceListResponse, error) {
	log := logrus.WithFields(logrus.Fields{
		"diskType": req.DiskType,
		"diskName": req.DiskName,
	})

	log.Trace("Disk Server: Listing disk replica instances")

	if req.DiskName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "disk name is required")
	}

	ops, ok := s.ops[req.DiskType]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported disk type %v", req.DiskType)
	}
	return ops.DiskReplicaInstanceList(req)
}

func (ops FilesystemDiskOps) DiskReplicaInstanceList(req *rpc.DiskReplicaInstanceListRequest) (*rpc.DiskReplicaInstanceListResponse, error) {
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported disk type %v", req.DiskType)
}

func (ops BlockDiskOps) DiskReplicaInstanceList(req *rpc.DiskReplicaInstanceListRequest) (*rpc.DiskReplicaInstanceListResponse, error) {
	replicas, err := ops.spdkClient.ReplicaList()
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, err.Error())
	}
	instances := map[string]*rpc.ReplicaInstance{}
	for name, replica := range replicas {
		instances[name] = replicaToReplicaInstance(replica)
	}
	return &rpc.DiskReplicaInstanceListResponse{
		ReplicaInstances: instances,
	}, nil
}

func (s *Server) DiskReplicaInstanceDelete(ctx context.Context, req *rpc.DiskReplicaInstanceDeleteRequest) (*emptypb.Empty, error) {
	log := logrus.WithFields(logrus.Fields{
		"diskType":            req.DiskType,
		"diskName":            req.DiskName,
		"diskUUID":            req.DiskUuid,
		"replciaInstanceName": req.ReplciaInstanceName,
	})

	log.Info("Disk Server: Deleting disk replica instance")

	if req.DiskName == "" || req.DiskUuid == "" || req.ReplciaInstanceName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "disk name, disk UUID and replica instance name are required")
	}

	ops, ok := s.ops[req.DiskType]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported disk type %v", req.DiskType)
	}
	return ops.DiskReplicaInstanceDelete(req)
}

func (ops FilesystemDiskOps) DiskReplicaInstanceDelete(req *rpc.DiskReplicaInstanceDeleteRequest) (*emptypb.Empty, error) {
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported disk type %v", req.DiskType)
}

func (ops BlockDiskOps) DiskReplicaInstanceDelete(req *rpc.DiskReplicaInstanceDeleteRequest) (*emptypb.Empty, error) {
	err := ops.spdkClient.ReplicaDelete(req.ReplciaInstanceName, true)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, err.Error())
	}
	return &emptypb.Empty{}, nil
}

func (s *Server) MetricsGet(ctx context.Context, req *rpc.DiskGetRequest) (*rpc.DiskMetricsGetReply, error) {
	log := logrus.WithFields(logrus.Fields{
		"diskType": req.DiskType,
		"diskName": req.DiskName,
		"diskPath": req.DiskPath,
	})

	log.Trace("Disk Server: Getting disk metrics")

	if req.DiskName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "disk name is required")
	}

	ops, ok := s.ops[req.DiskType]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported disk type %v", req.DiskType)
	}
	return ops.MetricsGet(req)
}

func (ops FilesystemDiskOps) MetricsGet(req *rpc.DiskGetRequest) (*rpc.DiskMetricsGetReply, error) {
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported disk type %v", req.DiskType)
}

func (ops BlockDiskOps) MetricsGet(req *rpc.DiskGetRequest) (*rpc.DiskMetricsGetReply, error) {
	metrics, err := ops.spdkClient.MetricsGet(req.DiskName)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, err.Error())
	}

	// Convert SPDK metrics to ptypes.Metrics format
	return &rpc.DiskMetricsGetReply{
		Metrics: &enginerpc.Metrics{
			ReadThroughput:  metrics.ReadThroughput,
			WriteThroughput: metrics.WriteThroughput,
			ReadLatency:     metrics.ReadLatency,
			WriteLatency:    metrics.WriteLatency,
			ReadIOPS:        metrics.ReadIOPS,
			WriteIOPS:       metrics.WriteIOPS,
		},
	}, nil
}

func spdkDiskToDisk(disk *spdkrpc.Disk) *rpc.Disk {
	return &rpc.Disk{
		Id:          disk.Id,
		Name:        disk.Name,
		Uuid:        disk.Uuid,
		Path:        disk.Path,
		Type:        disk.Type,
		Driver:      disk.Driver,
		TotalSize:   disk.TotalSize,
		FreeSize:    disk.FreeSize,
		TotalBlocks: disk.TotalBlocks,
		FreeBlocks:  disk.FreeBlocks,
		BlockSize:   disk.BlockSize,
		ClusterSize: disk.ClusterSize,
		State:       disk.State,
	}
}

func replicaToReplicaInstance(r *api.Replica) *rpc.ReplicaInstance {
	return &rpc.ReplicaInstance{
		Name:       r.Name,
		DiskName:   r.LvsName,
		DiskUuid:   r.LvsUUID,
		SpecSize:   r.SpecSize,
		ActualSize: r.ActualSize,
	}
}
</file>

<file path="pkg/disk/healthchecker.go">
package disk

import (
	"time"

	"github.com/sirupsen/logrus"

	"github.com/longhorn/longhorn-instance-manager/pkg/types"
	"github.com/longhorn/longhorn-instance-manager/pkg/util"
)

type HealthChecker interface {
	IsRunning(address string) bool
	WaitForRunning(address, name string, stopCh chan struct{}) bool
}

type GRPCHealthChecker struct{}

func (c *GRPCHealthChecker) IsRunning(address string) bool {
	return util.GRPCServiceReadinessProbe(address)
}

func (c *GRPCHealthChecker) WaitForRunning(address, name string, stopCh chan struct{}) bool {
	ticker := time.NewTicker(types.WaitInterval)
	defer ticker.Stop()

	for i := 0; i < types.WaitCount; i++ {
		select {
		case <-stopCh:
			logrus.Infof("Stop waiting for gRPC service of disk service %v to start at %v", name, address)
			return false

		case <-ticker.C:
			if c.IsRunning(address) {
				logrus.Infof("Disk service %v has started at %v", name, address)
				return true
			}
			logrus.Infof("Wait for gRPC service of disk service %v to start at %v", name, address)
		}
	}

	return false
}
</file>

<file path="pkg/disk/types.go">
package disk

const (
	DiskTypeFilesystem = "filesystem"
	DiskTypeBlock      = "block"
)
</file>

<file path="pkg/health/disk_service_health_probe.go">
package health

import (
	"context"
	"fmt"
	"time"

	"github.com/sirupsen/logrus"
	healthpb "google.golang.org/grpc/health/grpc_health_v1"

	"github.com/longhorn/longhorn-instance-manager/pkg/disk"
	"github.com/longhorn/longhorn-instance-manager/pkg/types"
)

type CheckDiskServer struct {
	server *disk.Server
}

func NewDiskHealthCheckServer(server *disk.Server) *CheckDiskServer {
	return &CheckDiskServer{
		server: server,
	}
}

func (hc *CheckDiskServer) Check(context.Context, *healthpb.HealthCheckRequest) (*healthpb.HealthCheckResponse, error) {
	if hc.server != nil {
		return &healthpb.HealthCheckResponse{
			Status: healthpb.HealthCheckResponse_SERVING,
		}, nil
	}

	return &healthpb.HealthCheckResponse{
		Status: healthpb.HealthCheckResponse_NOT_SERVING,
	}, fmt.Errorf("server or instance manager is not running")
}

func (hc *CheckDiskServer) Watch(req *healthpb.HealthCheckRequest, ws healthpb.Health_WatchServer) error {
	for {
		if hc.server != nil {
			if err := ws.Send(&healthpb.HealthCheckResponse{
				Status: healthpb.HealthCheckResponse_SERVING,
			}); err != nil {
				logrus.WithError(err).Errorf("Failed to send health check result %v for %s",
					healthpb.HealthCheckResponse_SERVING, types.DiskGrpcService)
			}
		} else {
			if err := ws.Send(&healthpb.HealthCheckResponse{
				Status: healthpb.HealthCheckResponse_NOT_SERVING,
			}); err != nil {
				logrus.WithError(err).Errorf("Failed to send health check result %v for %s",
					healthpb.HealthCheckResponse_NOT_SERVING, types.DiskGrpcService)
			}

		}
		time.Sleep(time.Second)
	}
}

func (hc *CheckDiskServer) List(context.Context, *healthpb.HealthListRequest) (*healthpb.HealthListResponse, error) {
	return &healthpb.HealthListResponse{
		Statuses: map[string]*healthpb.HealthCheckResponse{
			"grpc": {
				Status: healthpb.HealthCheckResponse_SERVING,
			},
		},
	}, nil
}
</file>

<file path="pkg/health/health_probe.go">
package health

import (
	"context"
	"fmt"
	"time"

	"github.com/sirupsen/logrus"

	healthpb "google.golang.org/grpc/health/grpc_health_v1"

	"github.com/longhorn/longhorn-instance-manager/pkg/process"
)

type CheckServer struct {
	pl *process.Manager
}

func NewHealthCheckServer(pl *process.Manager) *CheckServer {
	return &CheckServer{
		pl: pl,
	}
}

func (hc *CheckServer) Check(context.Context, *healthpb.HealthCheckRequest) (*healthpb.HealthCheckResponse, error) {
	if hc.pl != nil {
		return &healthpb.HealthCheckResponse{
			Status: healthpb.HealthCheckResponse_SERVING,
		}, nil
	}

	return &healthpb.HealthCheckResponse{
		Status: healthpb.HealthCheckResponse_NOT_SERVING,
	}, fmt.Errorf("engine Manager or Process Manager or Instance Manager is not running")
}

func (hc *CheckServer) Watch(req *healthpb.HealthCheckRequest, ws healthpb.Health_WatchServer) error {
	for {
		if hc.pl != nil {
			if err := ws.Send(&healthpb.HealthCheckResponse{
				Status: healthpb.HealthCheckResponse_SERVING,
			}); err != nil {
				logrus.WithError(err).Errorf("Failed to send health check result %v for gRPC process management server",
					healthpb.HealthCheckResponse_SERVING)
			}
		} else {
			if err := ws.Send(&healthpb.HealthCheckResponse{
				Status: healthpb.HealthCheckResponse_NOT_SERVING,
			}); err != nil {
				logrus.WithError(err).Errorf("Failed to send health check result %v for gRPC process management server",
					healthpb.HealthCheckResponse_NOT_SERVING)
			}
		}
		time.Sleep(time.Second)
	}
}

func (hc *CheckServer) List(context.Context, *healthpb.HealthListRequest) (*healthpb.HealthListResponse, error) {
	return &healthpb.HealthListResponse{
		Statuses: map[string]*healthpb.HealthCheckResponse{
			"grpc": {
				Status: healthpb.HealthCheckResponse_SERVING,
			},
		},
	}, nil
}
</file>

<file path="pkg/health/instance_service_health_probe.go">
package health

import (
	"context"
	"fmt"
	"time"

	"github.com/sirupsen/logrus"

	healthpb "google.golang.org/grpc/health/grpc_health_v1"

	"github.com/longhorn/longhorn-instance-manager/pkg/instance"
	"github.com/longhorn/longhorn-instance-manager/pkg/types"
)

type CheckInstanceServer struct {
	server *instance.Server
}

func NewInstanceHealthCheckServer(server *instance.Server) *CheckInstanceServer {
	return &CheckInstanceServer{
		server: server,
	}
}

func (hc *CheckInstanceServer) Check(context.Context, *healthpb.HealthCheckRequest) (*healthpb.HealthCheckResponse, error) {
	if hc.server != nil {
		return &healthpb.HealthCheckResponse{
			Status: healthpb.HealthCheckResponse_SERVING,
		}, nil
	}

	return &healthpb.HealthCheckResponse{
		Status: healthpb.HealthCheckResponse_NOT_SERVING,
	}, fmt.Errorf("server or instance manager is not running")
}

func (hc *CheckInstanceServer) Watch(req *healthpb.HealthCheckRequest, ws healthpb.Health_WatchServer) error {
	for {
		if hc.server != nil {
			if err := ws.Send(&healthpb.HealthCheckResponse{
				Status: healthpb.HealthCheckResponse_SERVING,
			}); err != nil {
				logrus.WithError(err).Errorf("Failed to send health check result %v for %s",
					healthpb.HealthCheckResponse_SERVING, types.InstanceGrpcService)
			}
		} else {
			if err := ws.Send(&healthpb.HealthCheckResponse{
				Status: healthpb.HealthCheckResponse_NOT_SERVING,
			}); err != nil {
				logrus.WithError(err).Errorf("Failed to send health check result %v for %s",
					healthpb.HealthCheckResponse_NOT_SERVING, types.InstanceGrpcService)
			}

		}
		time.Sleep(time.Second)
	}
}

func (hc *CheckInstanceServer) List(context.Context, *healthpb.HealthListRequest) (*healthpb.HealthListResponse, error) {
	return &healthpb.HealthListResponse{
		Statuses: map[string]*healthpb.HealthCheckResponse{
			"grpc": {
				Status: healthpb.HealthCheckResponse_SERVING,
			},
		},
	}, nil
}
</file>

<file path="pkg/health/proxy_health_probe.go">
package health

import (
	"context"
	"fmt"
	"time"

	"github.com/sirupsen/logrus"

	healthpb "google.golang.org/grpc/health/grpc_health_v1"

	"github.com/longhorn/longhorn-instance-manager/pkg/proxy"
	"github.com/longhorn/longhorn-instance-manager/pkg/types"
)

type CheckProxyServer struct {
	proxy *proxy.Proxy
}

func NewProxyHealthCheckServer(proxy *proxy.Proxy) *CheckProxyServer {
	return &CheckProxyServer{
		proxy: proxy,
	}
}

func (hc *CheckProxyServer) Check(context.Context, *healthpb.HealthCheckRequest) (*healthpb.HealthCheckResponse, error) {
	if hc.proxy != nil {
		return &healthpb.HealthCheckResponse{
			Status: healthpb.HealthCheckResponse_SERVING,
		}, nil
	}

	return &healthpb.HealthCheckResponse{
		Status: healthpb.HealthCheckResponse_NOT_SERVING,
	}, fmt.Errorf("proxy or instance manager is not running")
}

func (hc *CheckProxyServer) Watch(req *healthpb.HealthCheckRequest, ws healthpb.Health_WatchServer) error {
	for {
		if hc.proxy != nil {
			if err := ws.Send(&healthpb.HealthCheckResponse{
				Status: healthpb.HealthCheckResponse_SERVING,
			}); err != nil {
				logrus.WithError(err).Errorf("Failed to send health check result %v for %s",
					healthpb.HealthCheckResponse_SERVING, types.ProxyGRPCService)
			}
		} else {
			if err := ws.Send(&healthpb.HealthCheckResponse{
				Status: healthpb.HealthCheckResponse_NOT_SERVING,
			}); err != nil {
				logrus.WithError(err).Errorf("Failed to send health check result %v for %s",
					healthpb.HealthCheckResponse_NOT_SERVING, types.ProxyGRPCService)
			}

		}
		time.Sleep(time.Second)
	}
}

func (hc *CheckProxyServer) List(context.Context, *healthpb.HealthListRequest) (*healthpb.HealthListResponse, error) {
	return &healthpb.HealthListResponse{
		Statuses: map[string]*healthpb.HealthCheckResponse{
			"grpc": {
				Status: healthpb.HealthCheckResponse_SERVING,
			},
		},
	}, nil
}
</file>

<file path="pkg/health/spdk_service_health_probe.go">
package health

import (
	"context"
	"fmt"
	"time"

	"github.com/sirupsen/logrus"

	healthpb "google.golang.org/grpc/health/grpc_health_v1"

	spdk "github.com/longhorn/longhorn-spdk-engine/pkg/spdk"

	"github.com/longhorn/longhorn-instance-manager/pkg/types"
)

type CheckSPDKServer struct {
	server *spdk.Server
}

func NewSPDKHealthCheckServer(server *spdk.Server) *CheckSPDKServer {
	return &CheckSPDKServer{
		server: server,
	}
}

func (hc *CheckSPDKServer) Check(context.Context, *healthpb.HealthCheckRequest) (*healthpb.HealthCheckResponse, error) {
	if hc.server != nil {
		return &healthpb.HealthCheckResponse{
			Status: healthpb.HealthCheckResponse_SERVING,
		}, nil
	}

	return &healthpb.HealthCheckResponse{
		Status: healthpb.HealthCheckResponse_NOT_SERVING,
	}, fmt.Errorf("server or instance manager is not running")
}

func (hc *CheckSPDKServer) Watch(req *healthpb.HealthCheckRequest, ws healthpb.Health_WatchServer) error {
	for {
		if hc.server != nil {
			if err := ws.Send(&healthpb.HealthCheckResponse{
				Status: healthpb.HealthCheckResponse_SERVING,
			}); err != nil {
				logrus.WithError(err).Errorf("Failed to send health check result %v for %s",
					healthpb.HealthCheckResponse_SERVING, types.SpdkGrpcService)
			}
		} else {
			if err := ws.Send(&healthpb.HealthCheckResponse{
				Status: healthpb.HealthCheckResponse_NOT_SERVING,
			}); err != nil {
				logrus.WithError(err).Errorf("Failed to send health check result %v for %s",
					healthpb.HealthCheckResponse_NOT_SERVING, types.SpdkGrpcService)
			}

		}
		time.Sleep(time.Second)
	}
}

func (hc *CheckSPDKServer) List(context.Context, *healthpb.HealthListRequest) (*healthpb.HealthListResponse, error) {
	return &healthpb.HealthListResponse{
		Statuses: map[string]*healthpb.HealthCheckResponse{
			"grpc": {
				Status: healthpb.HealthCheckResponse_SERVING,
			},
		},
	}, nil
}
</file>

<file path="pkg/instance/healthchecker.go">
package instance

import (
	"time"

	"github.com/sirupsen/logrus"

	"github.com/longhorn/longhorn-instance-manager/pkg/types"
	"github.com/longhorn/longhorn-instance-manager/pkg/util"
)

type HealthChecker interface {
	IsRunning(address string) bool
	WaitForRunning(address, name string, stopCh chan struct{}) bool
}

type GRPCHealthChecker struct{}

func (c *GRPCHealthChecker) IsRunning(address string) bool {
	return util.GRPCServiceReadinessProbe(address)
}

func (c *GRPCHealthChecker) WaitForRunning(address, name string, stopCh chan struct{}) bool {
	ticker := time.NewTicker(types.WaitInterval)
	defer ticker.Stop()

	for i := 0; i < types.WaitCount; i++ {
		select {
		case <-stopCh:
			logrus.Infof("Stopped waiting for gRPC service of instance service %v to start at %v", name, address)
			return false

		case <-ticker.C:
			if c.IsRunning(address) {
				logrus.Infof("Instance service %v has started at %v", name, address)
				return true
			}
			logrus.Infof("Waiting for gRPC service of instance service %v to start at %v", name, address)
		}
	}

	return false
}
</file>

<file path="pkg/instance/instance.go">
package instance

import (
	"context"
	"fmt"
	"io"
	"time"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"golang.org/x/sync/errgroup"
	"google.golang.org/protobuf/types/known/emptypb"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	lhLonghorn "github.com/longhorn/go-common-libs/longhorn"
	spdkapi "github.com/longhorn/longhorn-spdk-engine/pkg/api"
	spdkclient "github.com/longhorn/longhorn-spdk-engine/pkg/client"
	rpc "github.com/longhorn/types/pkg/generated/imrpc"

	"github.com/longhorn/longhorn-instance-manager/pkg/client"
	"github.com/longhorn/longhorn-instance-manager/pkg/meta"
	"github.com/longhorn/longhorn-instance-manager/pkg/types"
)

const (
	maxMonitorRetryCount     = 10
	monitorRetryPollInterval = 1 * time.Second
)

type InstanceOps interface {
	InstanceCreate(*rpc.InstanceCreateRequest) (*rpc.InstanceResponse, error)
	InstanceDelete(*rpc.InstanceDeleteRequest) (*rpc.InstanceResponse, error)
	InstanceGet(*rpc.InstanceGetRequest) (*rpc.InstanceResponse, error)
	InstanceList(map[string]*rpc.InstanceResponse) error
	InstanceReplace(*rpc.InstanceReplaceRequest) (*rpc.InstanceResponse, error)
	InstanceLog(*rpc.InstanceLogRequest, rpc.InstanceService_InstanceLogServer) error
	InstanceSuspend(*rpc.InstanceSuspendRequest) (*emptypb.Empty, error)
	InstanceResume(*rpc.InstanceResumeRequest) (*emptypb.Empty, error)
	InstanceSwitchOverTarget(*rpc.InstanceSwitchOverTargetRequest) (*emptypb.Empty, error)
	InstanceDeleteTarget(*rpc.InstanceDeleteTargetRequest) (*emptypb.Empty, error)

	LogSetLevel(context.Context, *rpc.LogSetLevelRequest) (*emptypb.Empty, error)
	LogSetFlags(context.Context, *rpc.LogSetFlagsRequest) (*emptypb.Empty, error)
	LogGetLevel(context.Context, *rpc.LogGetLevelRequest) (*rpc.LogGetLevelResponse, error)
	LogGetFlags(context.Context, *rpc.LogGetFlagsRequest) (*rpc.LogGetFlagsResponse, error)
}

type V1DataEngineInstanceOps struct {
	processManagerServiceAddress string
}
type V2DataEngineInstanceOps struct {
	spdkServiceAddress string
}

type Server struct {
	rpc.UnimplementedInstanceServiceServer
	ctx           context.Context
	logsDir       string
	HealthChecker HealthChecker

	v2DataEngineEnabled bool
	ops                 map[rpc.DataEngine]InstanceOps
}

func NewServer(ctx context.Context, logsDir, processManagerServiceAddress, spdkServiceAddress string, v2DataEngineEnabled bool) (*Server, error) {
	ops := map[rpc.DataEngine]InstanceOps{
		rpc.DataEngine_DATA_ENGINE_V1: V1DataEngineInstanceOps{
			processManagerServiceAddress: processManagerServiceAddress,
		},
		rpc.DataEngine_DATA_ENGINE_V2: V2DataEngineInstanceOps{
			spdkServiceAddress: spdkServiceAddress,
		},
	}

	s := &Server{
		ctx:                 ctx,
		logsDir:             logsDir,
		v2DataEngineEnabled: v2DataEngineEnabled,
		HealthChecker:       &GRPCHealthChecker{},
		ops:                 ops,
	}

	go s.startMonitoring()

	return s, nil
}

func (s *Server) startMonitoring() {
	<-s.ctx.Done()
	logrus.Infof("%s: stopped monitoring due to the context done", types.InstanceGrpcService)
}

func (s *Server) VersionGet(ctx context.Context, req *emptypb.Empty) (*rpc.VersionResponse, error) {
	v := meta.GetVersion()
	return &rpc.VersionResponse{
		Version:   v.Version,
		GitCommit: v.GitCommit,
		BuildDate: v.BuildDate,

		InstanceManagerAPIVersion:    int64(v.InstanceManagerAPIVersion),
		InstanceManagerAPIMinVersion: int64(v.InstanceManagerAPIMinVersion),

		InstanceManagerProxyAPIVersion:    int64(v.InstanceManagerProxyAPIVersion),
		InstanceManagerProxyAPIMinVersion: int64(v.InstanceManagerProxyAPIMinVersion),
	}, nil
}

func (s *Server) InstanceCreate(ctx context.Context, req *rpc.InstanceCreateRequest) (*rpc.InstanceResponse, error) {
	logrus.WithFields(logrus.Fields{
		"name":            req.Spec.Name,
		"type":            req.Spec.Type,
		"dataEngine":      req.Spec.DataEngine,
		"upgradeRequired": req.Spec.UpgradeRequired,
	}).Info("Creating instance")

	ops, ok := s.ops[req.Spec.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.Spec.DataEngine)
	}
	return ops.InstanceCreate(req)
}

func (ops V1DataEngineInstanceOps) InstanceCreate(req *rpc.InstanceCreateRequest) (*rpc.InstanceResponse, error) {
	if req.Spec.ProcessInstanceSpec == nil {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "ProcessInstanceSpec is required for longhorn data engine")
	}

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	pmClient, err := client.NewProcessManagerClient(ctx, cancel, "tcp://"+ops.processManagerServiceAddress, nil)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create ProcessManagerClient").Error())
	}

	defer func() {
		if closeErr := pmClient.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"name":            req.Spec.Name,
				"type":            req.Spec.Type,
				"dataEngine":      req.Spec.DataEngine,
				"upgradeRequired": req.Spec.UpgradeRequired,
			}).WithError(closeErr).Warn("Failed to close ProcessManager client")
		}
	}()

	process, err := pmClient.ProcessCreate(req.Spec.Name, req.Spec.ProcessInstanceSpec.Binary, int(req.Spec.PortCount), req.Spec.ProcessInstanceSpec.Args, req.Spec.PortArgs)
	if err != nil {
		return nil, err
	}
	return processResponseToInstanceResponse(process, req.Spec.Type), nil
}

func (ops V2DataEngineInstanceOps) InstanceCreate(req *rpc.InstanceCreateRequest) (*rpc.InstanceResponse, error) {
	c, err := spdkclient.NewSPDKClient(ops.spdkServiceAddress)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create SPDK client").Error())
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"name":            req.Spec.Name,
				"type":            req.Spec.Type,
				"dataEngine":      req.Spec.DataEngine,
				"upgradeRequired": req.Spec.UpgradeRequired,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	switch req.Spec.Type {
	case types.InstanceTypeEngine:
		engine, err := c.EngineCreate(req.Spec.Name, req.Spec.VolumeName, req.Spec.SpdkInstanceSpec.Frontend, req.Spec.SpdkInstanceSpec.Size, req.Spec.SpdkInstanceSpec.ReplicaAddressMap,
			req.Spec.PortCount, req.Spec.InitiatorAddress, req.Spec.TargetAddress, req.Spec.SpdkInstanceSpec.SalvageRequested, req.Spec.SpdkInstanceSpec.UblkQueueDepth,
			req.Spec.SpdkInstanceSpec.UblkNumberOfQueue)
		if err != nil {
			return nil, err
		}
		return engineResponseToInstanceResponse(engine), nil
	case types.InstanceTypeReplica:
		replica, err := c.ReplicaCreate(req.Spec.Name, req.Spec.SpdkInstanceSpec.DiskName, req.Spec.SpdkInstanceSpec.DiskUuid, req.Spec.SpdkInstanceSpec.Size, req.Spec.PortCount, req.Spec.SpdkInstanceSpec.BackingImageName)
		if err != nil {
			return nil, err
		}
		return replicaResponseToInstanceResponse(replica), nil
	default:
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "unknown instance type %v", req.Spec.Type)
	}
}

func (s *Server) InstanceDelete(ctx context.Context, req *rpc.InstanceDeleteRequest) (*rpc.InstanceResponse, error) {
	logrus.WithFields(logrus.Fields{
		"name":            req.Name,
		"uuid":            req.Uuid,
		"type":            req.Type,
		"dataEngine":      req.DataEngine,
		"diskUuid":        req.DiskUuid,
		"cleanupRequired": req.CleanupRequired,
	}).Info("Deleting instance")

	ops, ok := s.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.InstanceDelete(req)
}

func (ops V1DataEngineInstanceOps) InstanceDelete(req *rpc.InstanceDeleteRequest) (*rpc.InstanceResponse, error) {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	pmClient, err := client.NewProcessManagerClient(ctx, cancel, "tcp://"+ops.processManagerServiceAddress, nil)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create ProcessManagerClient").Error())
	}
	defer func() {
		if closeErr := pmClient.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"name":            req.Name,
				"uuid":            req.Uuid,
				"type":            req.Type,
				"dataEngine":      req.DataEngine,
				"diskUuid":        req.DiskUuid,
				"cleanupRequired": req.CleanupRequired,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	process, err := pmClient.ProcessDelete(req.Name, req.Uuid)
	if err != nil {
		return nil, err
	}
	return processResponseToInstanceResponse(process, req.Type), nil
}

func (ops V2DataEngineInstanceOps) InstanceDelete(req *rpc.InstanceDeleteRequest) (*rpc.InstanceResponse, error) {
	c, err := spdkclient.NewSPDKClient(ops.spdkServiceAddress)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create SPDK client").Error())
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"name":            req.Name,
				"uuid":            req.Uuid,
				"type":            req.Type,
				"dataEngine":      req.DataEngine,
				"diskUuid":        req.DiskUuid,
				"cleanupRequired": req.CleanupRequired,
			}).WithError(closeErr).Warn("Failed to close SPDK Client")
		}
	}()

	if req.Uuid != "" {
		logrus.Debugf("Deleting instance %v with UUID %v", req.Name, req.Uuid)
	}

	switch req.Type {
	case types.InstanceTypeEngine:
		if req.CleanupRequired {
			err = c.EngineDelete(req.Name)
		}
	case types.InstanceTypeReplica:
		err = c.ReplicaDelete(req.Name, req.CleanupRequired)
	default:
		err = grpcstatus.Errorf(grpccodes.InvalidArgument, "unknown instance type %v", req.Type)
	}
	if err != nil {
		return nil, err
	}

	return &rpc.InstanceResponse{
		Spec: &rpc.InstanceSpec{
			Name: req.Name,
		},
		Status: &rpc.InstanceStatus{
			State: types.ProcessStateStopped,
		},
		Deleted: true,
	}, nil
}

func (s *Server) InstanceGet(ctx context.Context, req *rpc.InstanceGetRequest) (*rpc.InstanceResponse, error) {
	logrus.WithFields(logrus.Fields{
		"name":       req.Name,
		"type":       req.Type,
		"dataEngine": req.DataEngine,
	}).Trace("Getting instance")

	ops, ok := s.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.InstanceGet(req)
}

func (ops V1DataEngineInstanceOps) InstanceGet(req *rpc.InstanceGetRequest) (*rpc.InstanceResponse, error) {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	pmClient, err := client.NewProcessManagerClient(ctx, cancel, "tcp://"+ops.processManagerServiceAddress, nil)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create ProcessManagerClient").Error())
	}
	defer func() {
		if closeErr := pmClient.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"name":       req.Name,
				"type":       req.Type,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to ProcessManager client")
		}
	}()

	process, err := pmClient.ProcessGet(req.Name)
	if err != nil {
		return nil, err
	}
	return processResponseToInstanceResponse(process, req.Type), nil
}

func (ops V2DataEngineInstanceOps) InstanceGet(req *rpc.InstanceGetRequest) (*rpc.InstanceResponse, error) {
	c, err := spdkclient.NewSPDKClient(ops.spdkServiceAddress)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create SPDK client").Error())
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"name":       req.Name,
				"type":       req.Type,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	switch req.Type {
	case types.InstanceTypeEngine:
		engine, err := c.EngineGet(req.Name)
		if err != nil {
			return nil, err
		}
		return engineResponseToInstanceResponse(engine), nil
	case types.InstanceTypeReplica:
		replica, err := c.ReplicaGet(req.Name)
		if err != nil {
			return nil, err
		}
		return replicaResponseToInstanceResponse(replica), nil
	default:
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "unknown instance type %v", req.Type)
	}
}

func (s *Server) InstanceList(ctx context.Context, req *emptypb.Empty) (*rpc.InstanceListResponse, error) {
	logrus.WithFields(logrus.Fields{}).Trace("Listing instances")

	instances := map[string]*rpc.InstanceResponse{}

	err := s.ops[rpc.DataEngine_DATA_ENGINE_V1].InstanceList(instances)
	if err != nil {
		return nil, err
	}

	if s.v2DataEngineEnabled {
		err := s.ops[rpc.DataEngine_DATA_ENGINE_V2].InstanceList(instances)
		if err != nil {
			return nil, err
		}
	}

	return &rpc.InstanceListResponse{
		Instances: instances,
	}, nil
}

func (ops V1DataEngineInstanceOps) InstanceList(instances map[string]*rpc.InstanceResponse) error {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	pmClient, err := client.NewProcessManagerClient(ctx, cancel, "tcp://"+ops.processManagerServiceAddress, nil)
	if err != nil {
		return grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create ProcessManagerClient").Error())
	}
	defer func() {
		if closeErr := pmClient.Close(); closeErr != nil {
			logrus.WithError(closeErr).Warn("Failed to close ProcessManager client")
		}
	}()

	processes, err := pmClient.ProcessList()
	if err != nil {
		return err
	}
	for _, process := range processes {
		processType := types.InstanceTypeReplica
		if lhLonghorn.IsEngineProcess(process.Spec.Name) {
			processType = types.InstanceTypeEngine
		}
		instances[process.Spec.Name] = processResponseToInstanceResponse(process, processType)
	}
	return nil
}

func (ops V2DataEngineInstanceOps) InstanceList(instances map[string]*rpc.InstanceResponse) error {
	c, err := spdkclient.NewSPDKClient(ops.spdkServiceAddress)
	if err != nil {
		return grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create SPDK client").Error())
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	replicas, err := c.ReplicaList()
	if err != nil {
		return err
	}
	for _, replica := range replicas {
		instances[replica.Name] = replicaResponseToInstanceResponse(replica)
	}

	engines, err := c.EngineList()
	if err != nil {
		return err
	}
	for _, engine := range engines {
		instances[engine.Name] = engineResponseToInstanceResponse(engine)
	}
	return nil
}

func (s *Server) InstanceReplace(ctx context.Context, req *rpc.InstanceReplaceRequest) (*rpc.InstanceResponse, error) {
	logrus.WithFields(logrus.Fields{
		"name":       req.Spec.Name,
		"type":       req.Spec.Type,
		"dataEngine": req.Spec.DataEngine,
	}).Info("Replacing instance")

	ops, ok := s.ops[req.Spec.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.Spec.DataEngine)
	}
	return ops.InstanceReplace(req)
}

func (ops V1DataEngineInstanceOps) InstanceReplace(req *rpc.InstanceReplaceRequest) (*rpc.InstanceResponse, error) {
	if req.Spec.ProcessInstanceSpec == nil {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "ProcessInstanceSpec is required for longhorn data engine")
	}

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	pmClient, err := client.NewProcessManagerClient(ctx, cancel, "tcp://"+ops.processManagerServiceAddress, nil)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create ProcessManagerClient").Error())
	}
	defer func() {
		if closeErr := pmClient.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"name":       req.Spec.Name,
				"type":       req.Spec.Type,
				"dataEngine": req.Spec.DataEngine,
			}).WithError(closeErr).Warn("Failed to close ProcessManager client")
		}
	}()

	process, err := pmClient.ProcessReplace(req.Spec.Name,
		req.Spec.ProcessInstanceSpec.Binary, int(req.Spec.PortCount), req.Spec.ProcessInstanceSpec.Args, req.Spec.PortArgs, req.TerminateSignal)
	if err != nil {
		return nil, err
	}

	return processResponseToInstanceResponse(process, req.Spec.Type), nil
}

func (ops V2DataEngineInstanceOps) InstanceReplace(req *rpc.InstanceReplaceRequest) (*rpc.InstanceResponse, error) {
	return nil, grpcstatus.Error(grpccodes.Unimplemented, "v2 data engine instance replace is not supported")
}

func (s *Server) InstanceLog(req *rpc.InstanceLogRequest, srv rpc.InstanceService_InstanceLogServer) error {
	logrus.WithFields(logrus.Fields{
		"name":       req.Name,
		"type":       req.Type,
		"dataEngine": req.DataEngine,
	}).Info("Getting instance log")

	ops, ok := s.ops[req.DataEngine]
	if !ok {
		return grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.InstanceLog(req, srv)
}

func (ops V1DataEngineInstanceOps) InstanceLog(req *rpc.InstanceLogRequest, srv rpc.InstanceService_InstanceLogServer) error {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	pmClient, err := client.NewProcessManagerClient(ctx, cancel, "tcp://"+ops.processManagerServiceAddress, nil)
	if err != nil {
		return grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create ProcessManagerClient").Error())
	}
	defer func() {
		if closeErr := pmClient.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"name":       req.Name,
				"type":       req.Type,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close ProcessManager client")
		}
	}()

	stream, err := pmClient.ProcessLog(context.Background(), req.Name)
	if err != nil {
		return err
	}
	for {
		line, err := stream.Recv()
		if err == io.EOF {
			break
		} else if err != nil {
			logrus.WithError(err).Error("Failed to receive log")
			return err
		}

		if err := srv.Send(&rpc.LogResponse{Line: line}); err != nil {
			return err
		}
	}
	return nil
}

func (ops V2DataEngineInstanceOps) InstanceLog(req *rpc.InstanceLogRequest, srv rpc.InstanceService_InstanceLogServer) error {
	return grpcstatus.Error(grpccodes.Unimplemented, "v2 data engine instance log is not supported")
}

func (s *Server) handleNotify(ctx context.Context, notifyChan chan struct{}, srv rpc.InstanceService_InstanceWatchServer) error {
	logrus.Info("Start handling notify")

	for {
		select {
		case <-ctx.Done():
			logrus.Info("Stopped handling notify due to the context done")
			return ctx.Err()
		case <-notifyChan:
			if err := srv.Send(&emptypb.Empty{}); err != nil {
				return errors.Wrap(err, "failed to send instance response")
			}
		}
	}
}

func (s *Server) InstanceWatch(req *emptypb.Empty, srv rpc.InstanceService_InstanceWatchServer) error {
	logrus.Info("Start watching instances")

	done := make(chan struct{})

	clients := map[string]interface{}{}
	go func() {
		<-done

		logrus.Info("Stopped clients for watching instances")
		for name, c := range clients {
			switch c := c.(type) {
			case *client.ProcessManagerClient:
				if closeErr := c.Close(); closeErr != nil {
					logrus.WithError(closeErr).Warn("Failed to close ProcessManager client")
				}
			case *spdkclient.SPDKClient:
				if closeErr := c.Close(); closeErr != nil {
					logrus.WithError(closeErr).Warn("Failed to close SPDK client")
				}
			}
			delete(clients, name)
		}
		close(done)
	}()

	// Create a client for watching processes
	ops := s.ops[rpc.DataEngine_DATA_ENGINE_V1].(V1DataEngineInstanceOps)
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	pmClient, err := client.NewProcessManagerClient(ctx, cancel, "tcp://"+ops.processManagerServiceAddress, nil)
	if err != nil {
		done <- struct{}{}
		return grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create ProcessManagerClient").Error())
	}
	clients["processManagerClient"] = pmClient

	var spdkClient *spdkclient.SPDKClient
	if s.v2DataEngineEnabled {
		// Create a client for watching SPDK engines and replicas
		ops := s.ops[rpc.DataEngine_DATA_ENGINE_V2].(V2DataEngineInstanceOps)
		spdkClient, err = spdkclient.NewSPDKClient(ops.spdkServiceAddress)
		if err != nil {
			done <- struct{}{}
			return grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create SPDK client").Error())
		}
		clients["spdkClient"] = spdkClient
	}

	notifyChan := make(chan struct{}, 1024)
	defer close(notifyChan)

	g, ctx := errgroup.WithContext(s.ctx)

	g.Go(func() error {
		defer func() {
			// Close the clients for closing streams and unblocking notifier Recv() with error.
			done <- struct{}{}
		}()
		err := s.handleNotify(ctx, notifyChan, srv)
		if err != nil {
			logrus.WithError(err).Error("Failed to handle notify")
		}
		return err
	})

	g.Go(func() error {
		return s.watchProcess(ctx, req, pmClient, notifyChan)
	})

	if s.v2DataEngineEnabled {
		g.Go(func() error {
			return s.watchSPDKEngine(ctx, req, spdkClient, notifyChan)
		})

		g.Go(func() error {
			return s.watchSPDKReplica(ctx, req, spdkClient, notifyChan)
		})
	}

	if err := g.Wait(); err != nil {
		logrus.WithError(err).Error("Failed to watch instances")
		return errors.Wrap(err, "failed to watch instances")
	}

	return nil
}

func (s *Server) watchSPDKReplica(ctx context.Context, req *emptypb.Empty, client *spdkclient.SPDKClient, notifyChan chan struct{}) error {
	logrus.Info("Start watching SPDK replicas")

	notifier, err := client.ReplicaWatch(context.Background())
	if err != nil {
		return errors.Wrap(err, "failed to create SPDK replica watch notifier")
	}

	failureCount := 0
	for {
		if failureCount >= maxMonitorRetryCount {
			logrus.Errorf("Continuously receiving errors for %v times, stopping watching SPDK replicas", maxMonitorRetryCount)
			return fmt.Errorf("continuously receiving errors for %v times, stopping watching SPDK replicas", maxMonitorRetryCount)
		}

		select {
		case <-ctx.Done():
			logrus.Info("Stopped watching SPDK replicas")
			return ctx.Err()
		default:
			_, err := notifier.Recv()
			if err != nil {
				status, ok := grpcstatus.FromError(err)
				if ok && status.Code() == grpccodes.Canceled {
					logrus.WithError(err).Warn("SPDK replica watch is canceled")
					return err
				}
				logrus.WithError(err).Error("Failed to receive next item in SPDK replica watch")
				time.Sleep(monitorRetryPollInterval)
				failureCount++
			} else {
				notifyChan <- struct{}{}
			}
		}
	}
}

func (s *Server) watchSPDKEngine(ctx context.Context, req *emptypb.Empty, client *spdkclient.SPDKClient, notifyChan chan struct{}) error {
	logrus.Info("Start watching SPDK engines")

	notifier, err := client.EngineWatch(context.Background())
	if err != nil {
		return errors.Wrap(err, "failed to create SPDK engine watch notifier")
	}

	failureCount := 0
	for {
		if failureCount >= maxMonitorRetryCount {
			logrus.Errorf("Continuously receiving errors for %v times, stopping watching SPDK engines", maxMonitorRetryCount)
			return fmt.Errorf("continuously receiving errors for %v times, stopping watching SPDK engines", maxMonitorRetryCount)
		}

		select {
		case <-ctx.Done():
			logrus.Info("Stopped watching SPDK engines")
			return ctx.Err()
		default:
			_, err := notifier.Recv()
			if err != nil {
				status, ok := grpcstatus.FromError(err)
				if ok && status.Code() == grpccodes.Canceled {
					logrus.WithError(err).Warn("SPDK engine watch is canceled")
					return err
				}
				logrus.WithError(err).Error("Failed to receive next item in SPDK engine watch")
				time.Sleep(monitorRetryPollInterval)
				failureCount++
			} else {
				notifyChan <- struct{}{}
			}
		}
	}
}

func (s *Server) watchProcess(ctx context.Context, req *emptypb.Empty, client *client.ProcessManagerClient, notifyChan chan struct{}) error {
	logrus.Info("Start watching processes")

	notifier, err := client.ProcessWatch(context.Background())
	if err != nil {
		return errors.Wrap(err, "failed to create process watch notifier")
	}

	failureCount := 0
	for {
		if failureCount >= maxMonitorRetryCount {
			logrus.Errorf("Continuously receiving errors for %v times, stopping watching processes", maxMonitorRetryCount)
			return fmt.Errorf("continuously receiving errors for %v times, stopping watching processes", maxMonitorRetryCount)
		}

		select {
		case <-ctx.Done():
			logrus.Info("Stopped watching processes")
			return ctx.Err()
		default:
			_, err := notifier.Recv()
			if err != nil {
				status, ok := grpcstatus.FromError(err)
				if ok && status.Code() == grpccodes.Canceled {
					logrus.WithError(err).Warn("Process watch is canceled")
					return err
				}
				logrus.WithError(err).Error("Failed to receive next item in process watch")
				time.Sleep(monitorRetryPollInterval)
				failureCount++
			} else {
				notifyChan <- struct{}{}
			}
		}
	}
}

func processResponseToInstanceResponse(p *rpc.ProcessResponse, processType string) *rpc.InstanceResponse {
	// v1 data engine doesn't support the separation of initiator and target, so
	// initiator and target are always on the same node.
	targetPortStart := int32(0)
	targetPortEnd := int32(0)
	if processType == types.InstanceTypeEngine {
		targetPortStart = p.Status.PortStart
		targetPortEnd = p.Status.PortEnd
	}
	return &rpc.InstanceResponse{
		Spec: &rpc.InstanceSpec{
			Name: p.Spec.Name,
			Type: processType,
			// Deprecated
			BackendStoreDriver: rpc.BackendStoreDriver_v1,
			DataEngine:         rpc.DataEngine_DATA_ENGINE_V1,
			ProcessInstanceSpec: &rpc.ProcessInstanceSpec{
				Binary: p.Spec.Binary,
				Args:   p.Spec.Args,
			},
			PortCount: int32(p.Spec.PortCount),
			PortArgs:  p.Spec.PortArgs,
		},
		Status: &rpc.InstanceStatus{
			State:           p.Status.State,
			PortStart:       p.Status.PortStart,
			PortEnd:         p.Status.PortEnd,
			TargetPortStart: targetPortStart,
			TargetPortEnd:   targetPortEnd,
			ErrorMsg:        p.Status.ErrorMsg,
			Conditions:      p.Status.Conditions,
			Uuid:            p.Status.Uuid,
		},
		Deleted: p.Deleted,
	}
}

func replicaResponseToInstanceResponse(r *spdkapi.Replica) *rpc.InstanceResponse {
	return &rpc.InstanceResponse{
		Spec: &rpc.InstanceSpec{
			Name: r.Name,
			Type: types.InstanceTypeReplica,
			// Deprecated
			BackendStoreDriver: rpc.BackendStoreDriver_v2,
			DataEngine:         rpc.DataEngine_DATA_ENGINE_V2,
		},
		Status: &rpc.InstanceStatus{
			State:      r.State,
			ErrorMsg:   r.ErrorMsg,
			PortStart:  r.PortStart,
			PortEnd:    r.PortEnd,
			Conditions: make(map[string]bool),
			Uuid:       r.UUID,
		},
	}
}

func engineResponseToInstanceResponse(e *spdkapi.Engine) *rpc.InstanceResponse {
	return &rpc.InstanceResponse{
		Spec: &rpc.InstanceSpec{
			Name: e.Name,
			Type: types.InstanceTypeEngine,
			// Deprecated
			BackendStoreDriver: rpc.BackendStoreDriver_v2,
			DataEngine:         rpc.DataEngine_DATA_ENGINE_V2,
		},
		Status: &rpc.InstanceStatus{
			State:                  e.State,
			ErrorMsg:               e.ErrorMsg,
			PortStart:              e.Port,
			PortEnd:                e.Port,
			TargetPortStart:        e.TargetPort,
			TargetPortEnd:          e.TargetPort,
			StandbyTargetPortStart: e.StandbyTargetPort,
			StandbyTargetPortEnd:   e.StandbyTargetPort,
			Conditions:             make(map[string]bool),
			UblkId:                 e.UblkID,
			Uuid:                   e.UUID,
		},
	}
}

func (s *Server) InstanceSuspend(ctx context.Context, req *rpc.InstanceSuspendRequest) (*emptypb.Empty, error) {
	logrus.WithFields(logrus.Fields{
		"name":       req.Name,
		"type":       req.Type,
		"dataEngine": req.DataEngine,
	}).Info("Suspending instance")

	ops, ok := s.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.InstanceSuspend(req)
}

func (ops V1DataEngineInstanceOps) InstanceSuspend(req *rpc.InstanceSuspendRequest) (*emptypb.Empty, error) {
	return nil, grpcstatus.Error(grpccodes.Unimplemented, "v1 data engine instance suspend is not supported")
}

func (ops V2DataEngineInstanceOps) InstanceSuspend(req *rpc.InstanceSuspendRequest) (*emptypb.Empty, error) {
	c, err := spdkclient.NewSPDKClient(ops.spdkServiceAddress)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create SPDK client").Error())
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"name":       req.Name,
				"type":       req.Type,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	switch req.Type {
	case types.InstanceTypeEngine:
		err := c.EngineSuspend(req.Name)
		if err != nil {
			return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to suspend engine %v", req.Name).Error())
		}
		return &emptypb.Empty{}, nil
	case types.InstanceTypeReplica:
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "suspend is not supported for instance type %v", req.Type)
	default:
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "unknown instance type %v", req.Type)
	}
}

func (s *Server) InstanceResume(ctx context.Context, req *rpc.InstanceResumeRequest) (*emptypb.Empty, error) {
	logrus.WithFields(logrus.Fields{
		"name":       req.Name,
		"type":       req.Type,
		"dataEngine": req.DataEngine,
	}).Info("Resuming instance")

	ops, ok := s.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.InstanceResume(req)
}

func (ops V1DataEngineInstanceOps) InstanceResume(req *rpc.InstanceResumeRequest) (*emptypb.Empty, error) {
	return nil, grpcstatus.Error(grpccodes.Unimplemented, "v1 data engine instance resume is not supported")
}

func (ops V2DataEngineInstanceOps) InstanceResume(req *rpc.InstanceResumeRequest) (*emptypb.Empty, error) {
	c, err := spdkclient.NewSPDKClient(ops.spdkServiceAddress)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create SPDK client").Error())
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"name":       req.Name,
				"type":       req.Type,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	switch req.Type {
	case types.InstanceTypeEngine:
		err := c.EngineResume(req.Name)
		if err != nil {
			return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to resume engine %v", req.Name).Error())
		}
		return &emptypb.Empty{}, nil
	case types.InstanceTypeReplica:
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "resume is not supported for instance type %v", req.Type)
	default:
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "unknown instance type %v", req.Type)
	}
}

func (s *Server) InstanceSwitchOverTarget(ctx context.Context, req *rpc.InstanceSwitchOverTargetRequest) (*emptypb.Empty, error) {
	logrus.WithFields(logrus.Fields{
		"name":          req.Name,
		"type":          req.Type,
		"dataEngine":    req.DataEngine,
		"targetAddress": req.TargetAddress,
	}).Info("Switching over target instance")

	ops, ok := s.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.InstanceSwitchOverTarget(req)
}

func (ops V1DataEngineInstanceOps) InstanceSwitchOverTarget(req *rpc.InstanceSwitchOverTargetRequest) (*emptypb.Empty, error) {
	return nil, grpcstatus.Error(grpccodes.Unimplemented, "v1 data engine instance target switch over is not supported")
}

func (ops V2DataEngineInstanceOps) InstanceSwitchOverTarget(req *rpc.InstanceSwitchOverTargetRequest) (*emptypb.Empty, error) {
	c, err := spdkclient.NewSPDKClient(ops.spdkServiceAddress)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create SPDK client").Error())
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"name":          req.Name,
				"type":          req.Type,
				"dataEngine":    req.DataEngine,
				"targetAddress": req.TargetAddress,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	switch req.Type {
	case types.InstanceTypeEngine:
		err := c.EngineSwitchOverTarget(req.Name, req.TargetAddress)
		if err != nil {
			return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to switch over target for engine %v", req.Name).Error())
		}
		return &emptypb.Empty{}, nil
	case types.InstanceTypeReplica:
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "target switch over is not supported for instance type %v", req.Type)
	default:
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "unknown instance type %v", req.Type)
	}
}

func (s *Server) InstanceDeleteTarget(ctx context.Context, req *rpc.InstanceDeleteTargetRequest) (*emptypb.Empty, error) {
	logrus.WithFields(logrus.Fields{
		"name":       req.Name,
		"type":       req.Type,
		"dataEngine": req.DataEngine,
	}).Info("Deleting target")

	ops, ok := s.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.InstanceDeleteTarget(req)
}

func (ops V1DataEngineInstanceOps) InstanceDeleteTarget(req *rpc.InstanceDeleteTargetRequest) (*emptypb.Empty, error) {
	return nil, grpcstatus.Error(grpccodes.Unimplemented, "v1 data engine instance target delete is not supported")
}

func (ops V2DataEngineInstanceOps) InstanceDeleteTarget(req *rpc.InstanceDeleteTargetRequest) (*emptypb.Empty, error) {
	c, err := spdkclient.NewSPDKClient(ops.spdkServiceAddress)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create SPDK client").Error())
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"name":       req.Name,
				"type":       req.Type,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	switch req.Type {
	case types.InstanceTypeEngine:
		err := c.EngineDeleteTarget(req.Name)
		if err != nil {
			return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to delete target for engine %v", req.Name).Error())
		}
		return &emptypb.Empty{}, nil
	case types.InstanceTypeReplica:
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "target deletion is not supported for instance type %v", req.Type)
	default:
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "unknown instance type %v", req.Type)
	}
}
</file>

<file path="pkg/instance/log.go">
package instance

import (
	"context"
	"strings"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"google.golang.org/protobuf/types/known/emptypb"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	spdkclient "github.com/longhorn/longhorn-spdk-engine/pkg/client"
	rpc "github.com/longhorn/types/pkg/generated/imrpc"
)

const (
	NonSPDKLogLevelTrace = "TRACE"
	SPDKLogLevelDebug    = "DEBUG"
)

func (s *Server) LogSetLevel(ctx context.Context, req *rpc.LogSetLevelRequest) (resp *emptypb.Empty, err error) {
	ops, ok := s.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.LogSetLevel(ctx, req)
}

func logSetLevel(level string) error {
	// Set instance-manager log level.  We expect a string such as "debug", "info", or "warn".
	newLevel, err := logrus.ParseLevel(level)
	if err != nil {
		return err
	}

	oldLevel := logrus.GetLevel()
	if oldLevel != newLevel {
		logrus.Warnf("Updating log level from %v to %v", oldLevel, newLevel)
		logrus.SetLevel(newLevel)
	}

	return nil
}

// This method is used to set instance-manager internal log level regardless of engine type.
func (ops V1DataEngineInstanceOps) LogSetLevel(ctx context.Context, req *rpc.LogSetLevelRequest) (resp *emptypb.Empty, err error) {
	if err := logSetLevel(req.Level); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

// This method is used to set the log level for spdk_tgt, for v2 engine type.
func (ops V2DataEngineInstanceOps) LogSetLevel(ctx context.Context, req *rpc.LogSetLevelRequest) (resp *emptypb.Empty, err error) {
	spdkLevel := strings.ToUpper(req.Level)

	c, err := spdkclient.NewSPDKClient(ops.spdkServiceAddress)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create SPDK client").Error())
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	err = c.LogSetLevel(spdkLevel)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to set v2 data engine log level").Error())
	}

	return &emptypb.Empty{}, nil
}

func (s *Server) LogSetFlags(ctx context.Context, req *rpc.LogSetFlagsRequest) (resp *emptypb.Empty, err error) {
	ops, ok := s.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.LogSetFlags(ctx, req)
}

func (ops V1DataEngineInstanceOps) LogSetFlags(ctx context.Context, req *rpc.LogSetFlagsRequest) (resp *emptypb.Empty, err error) {
	// There is no V1 implementation.  Log flags are not a thing as they are for SPDK.
	return &emptypb.Empty{}, nil
}

func (ops V2DataEngineInstanceOps) LogSetFlags(ctx context.Context, req *rpc.LogSetFlagsRequest) (resp *emptypb.Empty, err error) {
	c, err := spdkclient.NewSPDKClient(ops.spdkServiceAddress)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create SPDK client").Error())
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	err = c.LogSetFlags(req.Flags)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to set log flags").Error())
	}
	return &emptypb.Empty{}, nil
}

func (s *Server) LogGetLevel(ctx context.Context, req *rpc.LogGetLevelRequest) (resp *rpc.LogGetLevelResponse, err error) {
	ops, ok := s.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.LogGetLevel(ctx, req)
}

func (ops V1DataEngineInstanceOps) LogGetLevel(ctx context.Context, req *rpc.LogGetLevelRequest) (resp *rpc.LogGetLevelResponse, err error) {
	return &rpc.LogGetLevelResponse{
		Level: logrus.GetLevel().String(),
	}, nil
}

func (ops V2DataEngineInstanceOps) LogGetLevel(ctx context.Context, req *rpc.LogGetLevelRequest) (resp *rpc.LogGetLevelResponse, err error) {
	return &rpc.LogGetLevelResponse{
		Level: logrus.GetLevel().String(),
	}, nil
}

func (s *Server) LogGetFlags(ctx context.Context, req *rpc.LogGetFlagsRequest) (resp *rpc.LogGetFlagsResponse, err error) {
	ops, ok := s.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.LogGetFlags(ctx, req)
}

func (ops V1DataEngineInstanceOps) LogGetFlags(ctx context.Context, req *rpc.LogGetFlagsRequest) (resp *rpc.LogGetFlagsResponse, err error) {
	// No implementation necessary.
	return &rpc.LogGetFlagsResponse{}, nil
}

func (ops V2DataEngineInstanceOps) LogGetFlags(ctx context.Context, req *rpc.LogGetFlagsRequest) (resp *rpc.LogGetFlagsResponse, err error) {
	c, err := spdkclient.NewSPDKClient(ops.spdkServiceAddress)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create SPDK client").Error())
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	flags, err := c.LogGetFlags()
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to get log flags").Error())
	}
	return &rpc.LogGetFlagsResponse{
		Flags: flags,
	}, nil
}
</file>

<file path="pkg/meta/version.go">
package meta

const (
	// InstanceManagerAPIVersion is used for compatibility check for longhorn-manager
	InstanceManagerAPIVersion    = 7
	InstanceManagerAPIMinVersion = 1

	// InstanceManagerProxyAPIVersion is used for compatibility check for longhorn-manager
	InstanceManagerProxyAPIVersion    = 6
	InstanceManagerProxyAPIMinVersion = 1

	// InstanceManagerDiskServiceAPIVersion used to communicate with the user e.g. longhorn-manager
	InstanceManagerDiskServiceAPIVersion    = 1
	InstanceManagerDiskServiceAPIMinVersion = 1
)

// Following variables are filled in by main.go
var (
	Version   string
	GitCommit string
	BuildDate string
)

type VersionOutput struct {
	Version   string `json:"version"`
	GitCommit string `json:"gitCommit"`
	BuildDate string `json:"buildDate"`

	InstanceManagerAPIVersion    int `json:"instanceManagerAPIVersion"`
	InstanceManagerAPIMinVersion int `json:"instanceManagerAPIMinVersion"`

	InstanceManagerProxyAPIVersion    int `json:"instanceManagerProxyAPIVersion"`
	InstanceManagerProxyAPIMinVersion int `json:"instanceManagerProxyAPIMinVersion"`
}

func GetVersion() VersionOutput {
	return VersionOutput{
		Version:   Version,
		GitCommit: GitCommit,
		BuildDate: BuildDate,

		InstanceManagerAPIVersion:    InstanceManagerAPIVersion,
		InstanceManagerAPIMinVersion: InstanceManagerAPIMinVersion,

		InstanceManagerProxyAPIVersion:    InstanceManagerProxyAPIVersion,
		InstanceManagerProxyAPIMinVersion: InstanceManagerProxyAPIMinVersion,
	}
}

type DiskServiceVersionOutput struct {
	Version   string `json:"version"`
	GitCommit string `json:"gitCommit"`
	BuildDate string `json:"buildDate"`

	InstanceManagerDiskServiceAPIVersion    int `json:"instanceManagerDiskServiceAPIVersion"`
	InstanceManagerDiskServiceAPIMinVersion int `json:"instanceManagerDiskServiceAPIMinVersion"`
}

func GetDiskServiceVersion() DiskServiceVersionOutput {
	return DiskServiceVersionOutput{
		Version:   Version,
		GitCommit: GitCommit,
		BuildDate: BuildDate,

		InstanceManagerDiskServiceAPIVersion:    InstanceManagerDiskServiceAPIVersion,
		InstanceManagerDiskServiceAPIMinVersion: InstanceManagerDiskServiceAPIMinVersion,
	}
}
</file>

<file path="pkg/process/command.go">
package process

import (
	"io"
	"os/exec"
	"path/filepath"
	"sync"
	"syscall"

	"github.com/sirupsen/logrus"
)

type Executor interface {
	NewCommand(name string, arg ...string) (Command, error)
}

type Command interface {
	Run() error
	SetOutput(io.Writer)
	IsRunning() bool
	Stop()
	StopWithSignal(signal syscall.Signal)
	Kill()
}

type BinaryExecutor struct{}

func (be *BinaryExecutor) NewCommand(name string, arg ...string) (Command, error) {
	return NewBinaryCommand(name, arg...)
}

type BinaryCommand struct {
	*sync.RWMutex
	*exec.Cmd
}

func NewBinaryCommand(binary string, arg ...string) (*BinaryCommand, error) {
	var err error

	binary, err = exec.LookPath(binary)
	if err != nil {
		return nil, err
	}

	binary, err = filepath.Abs(binary)
	if err != nil {
		return nil, err
	}

	cmd := exec.Command(binary, arg...)
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Pdeathsig: syscall.SIGKILL,
	}
	return &BinaryCommand{
		Cmd:     cmd,
		RWMutex: &sync.RWMutex{},
	}, nil
}

func (bc *BinaryCommand) SetOutput(writer io.Writer) {
	bc.Lock()
	defer bc.Unlock()
	bc.Stdout = writer
	bc.Stderr = writer
}

func (bc *BinaryCommand) IsRunning() bool {
	bc.RLock()
	defer bc.RUnlock()
	return bc.Process != nil && bc.ProcessState == nil
}

func (bc *BinaryCommand) StopWithSignal(signal syscall.Signal) {
	bc.RLock()
	defer bc.RUnlock()
	if bc.Process != nil {
		if err := bc.Process.Signal(signal); err != nil {
			logrus.WithError(err).Error("failed to send signal to process")
		}
	}
}

func (bc *BinaryCommand) Stop() {
	bc.RLock()
	defer bc.RUnlock()
	if bc.Process != nil {
		if err := bc.Process.Signal(syscall.SIGINT); err != nil {
			logrus.WithError(err).Error("failed to send signal to process")
		}
	}
}

func (bc *BinaryCommand) Kill() {
	bc.RLock()
	defer bc.RUnlock()
	if bc.Process != nil {
		if err := bc.Process.Signal(syscall.SIGKILL); err != nil {
			logrus.WithError(err).Error("failed to send signal to process")
		}
	}
}

type MockExecutor struct {
	CreationHook func(cmd *MockCommand) (*MockCommand, error)
}

func (me *MockExecutor) NewCommand(name string, arg ...string) (Command, error) {
	cmd := NewMockCommand(name, arg...)
	if me.CreationHook == nil {
		return cmd, nil
	}
	return me.CreationHook(NewMockCommand(name, arg...))
}

type MockCommand struct {
	*sync.RWMutex

	Binary string
	Args   []string

	stopCh chan error

	isRunning bool
	stopped   bool
}

func NewMockCommand(name string, arg ...string) *MockCommand {
	return &MockCommand{
		RWMutex: &sync.RWMutex{},

		Binary: name,
		Args:   arg,

		stopCh: make(chan error),

		isRunning: false,
		stopped:   false,
	}
}

func (mc *MockCommand) Run() error {
	mc.Lock()
	mc.isRunning = true
	mc.Unlock()

	return <-mc.stopCh
}

func (mc *MockCommand) SetOutput(writer io.Writer) {
}

func (mc *MockCommand) IsRunning() bool {
	mc.RLock()
	defer mc.RUnlock()
	return mc.isRunning
}

func (mc *MockCommand) Stop() {
	mc.Lock()
	mc.stopped = true
	mc.Unlock()

	mc.stopCh <- nil
}

func (mc *MockCommand) StopWithSignal(signal syscall.Signal) {
	mc.Stop()
}

func (mc *MockCommand) Kill() {
}
</file>

<file path="pkg/process/healthchecker.go">
package process

import (
	"time"

	"github.com/sirupsen/logrus"

	"github.com/longhorn/longhorn-instance-manager/pkg/types"
	"github.com/longhorn/longhorn-instance-manager/pkg/util"
)

type HealthChecker interface {
	IsRunning(address string) bool
	WaitForRunning(address, name string, stopCh chan struct{}) bool
}

type GRPCHealthChecker struct{}

func (c *GRPCHealthChecker) IsRunning(address string) bool {
	return util.GRPCServiceReadinessProbe(address)
}

func (c *GRPCHealthChecker) WaitForRunning(address, name string, stopCh chan struct{}) bool {
	ticker := time.NewTicker(types.WaitInterval)
	defer ticker.Stop()
	for i := 0; i < types.WaitCount; i++ {
		select {
		case <-stopCh:
			logrus.Infof("Stop waiting for gRPC service of process %v to start at %v", name, address)
			return false
		case <-ticker.C:
			if c.IsRunning(address) {
				logrus.Infof("Process %v has started at %v", name, address)
				return true
			}
			logrus.Infof("Wait for gRPC service of process %v to start at %v", name, address)
		}
	}
	return false
}

type MockHealthChecker struct{}

func (c *MockHealthChecker) IsRunning(address string) bool {
	return true
}

func (c *MockHealthChecker) WaitForRunning(address, name string, stopCh chan struct{}) bool {
	return true
}
</file>

<file path="pkg/process/process_manager.go">
package process

import (
	"context"
	"crypto/sha256"
	"encoding/hex"
	"fmt"
	"strconv"
	"strings"
	"sync"
	"syscall"
	"time"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
	"google.golang.org/protobuf/types/known/emptypb"
	"k8s.io/mount-utils"

	lhBitmap "github.com/longhorn/go-common-libs/bitmap"
	lhKubernetes "github.com/longhorn/go-common-libs/kubernetes"
	lhLonghorn "github.com/longhorn/go-common-libs/longhorn"
	rpc "github.com/longhorn/types/pkg/generated/imrpc"

	"github.com/longhorn/longhorn-instance-manager/pkg/types"
	"github.com/longhorn/longhorn-instance-manager/pkg/util"
	"github.com/longhorn/longhorn-instance-manager/pkg/util/broadcaster"
)

const (
	MountCheckInterval = 10 * time.Second

	DefaultEnginePortCount = 1
)

/* Lock order
   1. Manager.lock
   2. Process.lock
*/

type Manager struct {
	rpc.UnimplementedProcessManagerServiceServer
	ctx context.Context

	portRangeMin int32
	portRangeMax int32

	broadcaster *broadcaster.Broadcaster
	broadcastCh chan interface{}

	lock            *sync.RWMutex
	processes       map[string]*Process
	processUpdateCh chan *Process

	availablePorts *lhBitmap.Bitmap

	logsDir string

	Executor      Executor
	HealthChecker HealthChecker
}

func NewManager(ctx context.Context, portRange string, logsDir string) (*Manager, error) {
	start, end, err := ParsePortRange(portRange)
	if err != nil {
		return nil, err
	}
	bitmap, err := lhBitmap.NewBitmap(start, end)
	if err != nil {
		return nil, err
	}

	pm := &Manager{
		ctx:          ctx,
		portRangeMin: start,
		portRangeMax: end,

		broadcaster: &broadcaster.Broadcaster{},
		broadcastCh: make(chan interface{}),

		lock:            &sync.RWMutex{},
		processes:       map[string]*Process{},
		processUpdateCh: make(chan *Process),
		availablePorts:  bitmap,

		logsDir: logsDir,

		Executor:      &BinaryExecutor{},
		HealthChecker: &GRPCHealthChecker{},
	}
	// help to kickstart the broadcaster
	c, cancel := context.WithCancel(context.Background())
	defer cancel()
	if _, err := pm.broadcaster.Subscribe(c, pm.broadcastConnector); err != nil {
		return nil, err
	}
	go pm.startMonitoring()
	go pm.startInstanceConditionCheck()
	return pm, nil
}

func (pm *Manager) startMonitoring() {
	done := false

	for {
		select {
		case <-pm.ctx.Done():
			logrus.Infof("%s: stopped monitoring replicas due to the context done", types.ProcessManagerGrpcService)
			done = true
		case p := <-pm.processUpdateCh:
			resp := p.RPCResponse()
			pm.lock.RLock()
			// Modify response to indicate deletion.
			if _, exists := pm.processes[p.Name]; !exists {
				resp.Deleted = true
			}
			pm.lock.RUnlock()
			pm.broadcastCh <- interface{}(resp)
		}
		if done {
			break
		}
	}
}

func (pm *Manager) startInstanceConditionCheck() {
	done := false

	ticker := time.NewTicker(MountCheckInterval)
	defer ticker.Stop()

	for {
		select {
		case <-pm.ctx.Done():
			logrus.Infof("%s: stopped monitoring conditions due to the context done", types.ProcessManagerGrpcService)
			done = true
		case <-ticker.C:
			pm.checkMountPointStatusForEngine()
		}
		if done {
			break
		}
	}
}

func (pm *Manager) checkMountPointStatusForEngine() {
	volumeMountPointMap, err := util.GetVolumeMountPointMap()
	if err != nil {
		logrus.WithError(err).Warn("Failed to get all volume mount points")
	}
	// Locking is handled inside getProcessesToUpdateConditions.
	processesToUpdate := pm.getProcessesToUpdateConditions(volumeMountPointMap)
	for _, p := range processesToUpdate {
		p.UpdateCh <- p
	}
}

func (pm *Manager) getProcessesToUpdateConditions(volumeMountPointMap map[string]mount.MountPoint) []*Process {
	var processesToUpdate []*Process

	pm.lock.RLock()
	defer pm.lock.RUnlock()

	for _, p := range pm.processes {
		p.lock.Lock()
		if lhLonghorn.IsEngineProcess(p.Name) && p.State == StateRunning {
			volumeName := util.ProcessNameToVolumeName(p.Name)
			volumeNameSHA := sha256.Sum256([]byte(volumeName))
			volumeNameSHAStr := hex.EncodeToString(volumeNameSHA[:])

			if mp, exists := volumeMountPointMap[volumeNameSHAStr]; exists {
				p.Conditions[types.EngineConditionFilesystemReadOnly] = lhKubernetes.IsMountPointReadOnly(mp)
				processesToUpdate = append(processesToUpdate, p)
			}
		}
		p.lock.Unlock()
	}
	return processesToUpdate
}

// ProcessCreate will create a process according to the request.
// If the specified process name exists already, the creation will fail.
func (pm *Manager) ProcessCreate(ctx context.Context, req *rpc.ProcessCreateRequest) (ret *rpc.ProcessResponse, err error) {
	if req.Spec.Name == "" || req.Spec.Binary == "" {
		return nil, status.Errorf(codes.InvalidArgument, "missing required argument")
	}

	logrus.Infof("Process Manager: prepare to create process %v", req.Spec.Name)
	logger, err := util.NewLonghornWriter(req.Spec.Name, pm.logsDir)
	if err != nil {
		return nil, err
	}

	p := &Process{
		Name:      req.Spec.Name,
		Binary:    req.Spec.Binary,
		Args:      req.Spec.Args,
		PortCount: req.Spec.PortCount,
		PortArgs:  req.Spec.PortArgs,

		UUID: util.UUID(),

		State:      StateStarting,
		Conditions: make(map[string]bool),

		lock: &sync.RWMutex{},

		logger: logger,

		executor:      pm.Executor,
		healthChecker: pm.HealthChecker,
	}

	if err := pm.registerProcess(p); err != nil {
		return nil, err
	}

	p.UpdateCh <- p
	if err := p.Start(); err != nil {
		// initializing failed so we sent event about the failed state, but still return the process rpc below
		// this is to be consistent with the prior implementation
		logrus.WithError(err).Errorf("Process Manager: failed to init new process %v", req.Spec.Name)
		p.UpdateCh <- p
	} else {
		logrus.Infof("Process Manager: created process %v", req.Spec.Name)
	}

	return p.RPCResponse(), nil
}

// ProcessDelete will delete the process named by the request.
// If UUID is specified, the process will be deleted only if the UUID matches.
// If the process doesn't exist, the deletion will return with ErrorNotFound
func (pm *Manager) ProcessDelete(ctx context.Context, req *rpc.ProcessDeleteRequest) (ret *rpc.ProcessResponse, err error) {
	logrus.Infof("Process Manager: prepare to delete process %v, UUID %v", req.Name, req.Uuid)

	p := pm.findProcess(req.Name)
	if p == nil {
		return nil, status.Errorf(codes.NotFound, "cannot find process %v", req.Name)
	}
	if req.Uuid != "" && p.UUID != req.Uuid {
		return nil, status.Errorf(codes.NotFound, "cannot find process %v with UUID %v", req.Name, req.Uuid)
	}

	p.Stop()

	resp := p.RPCResponse()
	resp.Deleted = true

	pm.unregisterProcess(p)

	logrus.Infof("Process Manager: deleted process %v", req.Name)
	return resp, nil
}

func (pm *Manager) registerProcess(p *Process) error {
	pm.lock.Lock()
	defer pm.lock.Unlock()

	_, exists := pm.processes[p.Name]
	if exists {
		return status.Errorf(codes.AlreadyExists, "process %v already exists", p.Name)
	}

	if err := pm.allocateProcessPorts(p); err != nil {
		return err
	}

	p.UpdateCh = pm.processUpdateCh
	pm.processes[p.Name] = p

	return nil
}

func (pm *Manager) unregisterProcess(p *Process) {
	pm.lock.Lock()
	defer pm.lock.Unlock()

	// ProcessReplace call may change the process, need to ensure we're dealing with the right process
	if existingProcess, exists := pm.processes[p.Name]; !exists || existingProcess.UUID != p.UUID {
		return
	}

	go func() {
		for i := 0; i < types.WaitCount; i++ {
			if p.IsStopped() {
				break
			}
			logrus.Debugf("Process Manager: wait for process %v to shutdown before unregistering process", p.Name)
			time.Sleep(types.WaitInterval)
		}

		if !p.IsStopped() {
			logrus.Errorf("Process Manager: failed to unregister process %v since it is state %v rather than stopped", p.Name, p.State)
			return
		}

		func() {
			pm.lock.Lock()
			defer pm.lock.Unlock()
			if existingProcess, exists := pm.processes[p.Name]; !exists || existingProcess.UUID != p.UUID {
				return
			}

			delete(pm.processes, p.Name)
			pm.releaseProcessPorts(p)
		}()

		logrus.Infof("Process Manager: successfully unregistered process %v", p.Name)
		p.UpdateCh <- p
	}()
}

func (pm *Manager) findProcess(name string) *Process {
	pm.lock.RLock()
	defer pm.lock.RUnlock()

	return pm.processes[name]
}

// ProcessGet will get a process named by the request.
// If the process doesn't exist, the call will return with ErrorNotFound
func (pm *Manager) ProcessGet(ctx context.Context, req *rpc.ProcessGetRequest) (*rpc.ProcessResponse, error) {
	p := pm.findProcess(req.Name)
	if p == nil {
		return nil, status.Errorf(codes.NotFound, "cannot find process %v", req.Name)
	}

	return p.RPCResponse(), nil
}

func (pm *Manager) ProcessList(ctx context.Context, req *rpc.ProcessListRequest) (*rpc.ProcessListResponse, error) {
	pm.lock.RLock()
	defer pm.lock.RUnlock()

	resp := &rpc.ProcessListResponse{
		Processes: map[string]*rpc.ProcessResponse{},
	}
	for _, p := range pm.processes {
		resp.Processes[p.Name] = p.RPCResponse()
	}
	return resp, nil
}

func (pm *Manager) ProcessLog(req *rpc.LogRequest, srv rpc.ProcessManagerService_ProcessLogServer) error {
	logrus.Infof("Process Manager: start getting logs for process %v", req.Name)
	p := pm.findProcess(req.Name)
	if p == nil {
		return status.Errorf(codes.NotFound, "cannot find process %v", req.Name)
	}
	doneChan := make(chan struct{})
	logChan, err := p.logger.StreamLog(doneChan)
	if err != nil {
		return err
	}
	for logLine := range logChan {
		if err := srv.Send(&rpc.LogResponse{Line: logLine}); err != nil {
			doneChan <- struct{}{}
			close(doneChan)
			return err
		}
	}
	logrus.Infof("Process Manager: got logs for process %v", req.Name)
	return nil
}

func (pm *Manager) broadcastConnector() (chan interface{}, error) {
	return pm.broadcastCh, nil
}

func (pm *Manager) Subscribe() (<-chan interface{}, error) {
	return pm.broadcaster.Subscribe(context.TODO(), pm.broadcastConnector)
}

func (pm *Manager) ProcessWatch(req *emptypb.Empty, srv rpc.ProcessManagerService_ProcessWatchServer) (err error) {
	responseChan, err := pm.Subscribe()
	if err != nil {
		return err
	}

	defer func() {
		if err != nil {
			logrus.WithError(err).Error("Process manager update watch errored out")
		} else {
			logrus.Info("Process manager update watch ended successfully")
		}
	}()
	logrus.Info("Started new process manager update watch")

	for resp := range responseChan {
		r, ok := resp.(*rpc.ProcessResponse)
		if !ok {
			return fmt.Errorf("BUG: cannot get ProcessResponse from channel")
		}
		if err := srv.Send(r); err != nil {
			return err
		}
	}

	return nil
}

func (pm *Manager) allocatePorts(portCount int32) (int32, int32, error) {
	if portCount < 0 {
		return 0, 0, fmt.Errorf("invalid port count %v", portCount)
	}
	if portCount == 0 {
		return 0, 0, nil
	}
	start, end, err := pm.availablePorts.AllocateRange(portCount)
	if err != nil {
		return 0, 0, errors.Wrapf(err, "failed to allocate %v ports", portCount)
	}
	return int32(start), int32(end), nil
}

func (pm *Manager) releasePorts(start, end int32) error {
	if start < 0 || end < 0 {
		return fmt.Errorf("invalid start/end port %v %v", start, end)
	}
	return pm.availablePorts.ReleaseRange(start, end)
}

func ParsePortRange(portRange string) (int32, int32, error) {
	if portRange == "" {
		return 0, 0, fmt.Errorf("empty port range")
	}
	parts := strings.Split(portRange, "-")
	if len(parts) != 2 {
		return 0, 0, fmt.Errorf("invalid format for range: %s", portRange)
	}
	portStart, err := strconv.Atoi(strings.TrimSpace(parts[0]))
	if err != nil {
		return 0, 0, errors.Wrap(err, "invalid start port for range")
	}
	portEnd, err := strconv.Atoi(strings.TrimSpace(parts[1]))
	if err != nil {
		return 0, 0, errors.Wrap(err, "invalid end port for range")
	}
	return int32(portStart), int32(portEnd), nil
}

// ProcessReplace will replace a process with the new process according to the request.
// If the specified process name doesn't exist already, the replace will fail.
func (pm *Manager) ProcessReplace(ctx context.Context, req *rpc.ProcessReplaceRequest) (ret *rpc.ProcessResponse, err error) {
	if req.Spec.Name == "" || req.Spec.Binary == "" {
		return nil, status.Errorf(codes.InvalidArgument, "missing required argument")
	}
	if req.TerminateSignal != "SIGHUP" {
		return nil, status.Errorf(codes.InvalidArgument, "doesn't support terminate signal %v", req.TerminateSignal)
	}
	terminateSignal := syscall.SIGHUP

	logrus.Infof("Process Manager: prepare to replace process %v", req.Spec.Name)
	logger, err := util.NewLonghornWriter(req.Spec.Name, pm.logsDir)
	if err != nil {
		return nil, err
	}

	p := &Process{
		Name:      req.Spec.Name,
		Binary:    req.Spec.Binary,
		Args:      req.Spec.Args,
		PortCount: req.Spec.PortCount,
		PortArgs:  req.Spec.PortArgs,

		UUID: util.UUID(),

		State:      StateStarting,
		Conditions: make(map[string]bool),

		lock: &sync.RWMutex{},

		logger: logger,

		executor:      pm.Executor,
		healthChecker: pm.HealthChecker,
	}

	processToReplace, err := pm.initProcessReplace(p)
	if err != nil {
		return nil, err
	}

	if processToReplace.Binary == p.Binary {
		logrus.Infof("Process Manager: the existing process already has the updated engine image %v", p.Binary)
		return processToReplace.RPCResponse(), nil
	}

	cleanupReplacementProcess := func() {
		// TODO process ports should be tied to process UUID's right now only the port ranges is used
		//  so if one is not careful with allocation/release it's possible that different processes nuke each
		//  others ports
		p.Stop()
		pm.releaseProcessPorts(p)
		logrus.Errorf("Process Manager: cleaned up the replacement process %v with UUID %v", req.Spec.Name, p.UUID)
	}

	if err := p.Start(); err != nil {
		// initializing failed replacement process cleanup happens below
		logrus.WithError(err).Errorf("Process Manager: failed to init replacement process %v", req.Spec.Name)
		cleanupReplacementProcess()
		return nil, fmt.Errorf("failed to init replacement process %v", p.Name)
	}

	logrus.Infof("Process Manager: initiated replacement process %v with UUID %v", req.Spec.Name, p.UUID)
	for i := 0; i < 30; i++ {
		resp := p.RPCResponse()
		if resp.Status.State == types.ProcessStateRunning {
			logrus.Infof("Process Manager: replacement process for %v started running", req.Spec.Name)
			break
		} else if resp.Status.State != types.ProcessStateStarting {
			logrus.Errorf("Process Manager: replacement process for %v failed to start, now in state %v", req.Spec.Name, resp.Status.State)
			cleanupReplacementProcess()
			return nil, fmt.Errorf("failed to start replacement process %v", p.Name)
		}
		logrus.Debugf("Process Manager: waiting for the replace process %v to start", req.Spec.Name)
		time.Sleep(1 * time.Second)
	}

	// cleanup the process to replace this should always be safe to call outside of a lock
	processToReplace.StopWithSignal(terminateSignal)

	// we need to lock the evaluation & assignment
	// to be able to handle concurrent replace process calls for the same process
	pm.lock.Lock()
	if existingProcess, exists := pm.processes[p.Name]; !exists {
		logrus.Warnf("Process Manager: process %v with UUID %v no longer exists for replacement",
			p.Name, processToReplace.UUID)
	} else if existingProcess.UUID == processToReplace.UUID {
		pm.releaseProcessPorts(processToReplace)
		logrus.Infof("Process Manager: successfully unregistered old process %v", p.Name)
	} else {
		pm.lock.Unlock()
		logrus.Warnf("Process Manager: replace process %v the process to replace with UUID %v must have already been replaced found process with UUID %v cleaning up replacement process with UUID %v",
			p.Name, processToReplace.UUID, existingProcess.UUID, p.UUID)
		cleanupReplacementProcess()
		return nil, status.Errorf(codes.AlreadyExists, "process %v to replace has changed in the meantime", p.Name)
	}

	pm.processes[p.Name] = p
	logrus.Infof("Process Manager: process %v successfully registered replacement with UUID %v", p.Name, p.UUID)
	pm.lock.Unlock()

	p.UpdateCh <- p
	logrus.Infof("Process Manager: successfully replaced process %v", req.Spec.Name)
	return p.RPCResponse(), nil
}

func (pm *Manager) initProcessReplace(p *Process) (*Process, error) {
	pm.lock.Lock()
	defer pm.lock.Unlock()

	oldProcess, exists := pm.processes[p.Name]
	if !exists {
		return nil, status.Errorf(codes.NotFound, "existing process %v doesn't exists", p.Name)
	}

	if err := pm.allocateProcessPorts(p); err != nil {
		return nil, err
	}

	p.UpdateCh = pm.processUpdateCh
	return oldProcess, nil
}

func (pm *Manager) allocateProcessPorts(p *Process) error {
	var err error
	if len(p.PortArgs) > int(p.PortCount) {
		return fmt.Errorf("too many port args %v for port count %v", p.PortArgs, p.PortCount)
	}

	p.PortStart, p.PortEnd, err = pm.allocatePorts(p.PortCount)
	if err != nil {
		return errors.Wrapf(err, "cannot allocate %v ports for %v", p.PortCount, p.Name)
	}

	if len(p.PortArgs) != 0 {
		for i, arg := range p.PortArgs {
			if p.PortStart+int32(i) > p.PortEnd {
				return fmt.Errorf("cannot fit port args %v", arg)
			}
			p.Args = append(p.Args, strings.Split(arg+strconv.Itoa(int(p.PortStart)+i), ",")...)
		}
	}

	return nil
}

func (pm *Manager) releaseProcessPorts(p *Process) {
	if err := pm.releasePorts(p.PortStart, p.PortEnd); err != nil {
		logrus.WithError(err).Errorf("Process Manager: cannot deallocate %v ports (%v-%v) for %v",
			p.PortCount, p.PortStart, p.PortEnd, p.Name)
	}
}
</file>

<file path="pkg/process/process_test.go">
package process

import (
	"context"
	"os"
	"os/exec"
	"strconv"
	"sync"
	"testing"
	"time"

	rpc "github.com/longhorn/types/pkg/generated/imrpc"
	"github.com/sirupsen/logrus"
	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
	. "gopkg.in/check.v1"

	"github.com/longhorn/longhorn-instance-manager/pkg/types"
)

const (
	RetryCount        = 50
	RetryInterval     = 100 * time.Millisecond
	TestBinary        = "/engine-binaries/test/longhorn"
	TestBinaryMissing = "/engine-binaries/test-missing/longhorn"
	TestBinaryReplace = "/engine-binaries/test-replacement/longhorn"
)

func Test(t *testing.T) { TestingT(t) }

type TestSuite struct {
	shutdownCh chan error
	pm         *Manager
	logDir     string
}

var _ = Suite(&TestSuite{})

type ProcessWatcher struct {
	grpc.ServerStream
}

func (pw *ProcessWatcher) Send(resp *rpc.ProcessResponse) error {
	//Do nothing for now, just act as the receiving end
	return nil
}

func (s *TestSuite) SetUpSuite(c *C) {
	var err error

	logrus.SetLevel(logrus.DebugLevel)
	s.shutdownCh = make(chan error)

	s.logDir = os.TempDir()
	s.pm, err = NewManager(context.Background(), "10000-30000", s.logDir)
	c.Assert(err, IsNil)
	s.pm.Executor = &MockExecutor{
		CreationHook: func(cmd *MockCommand) (*MockCommand, error) {
			if cmd.Binary == TestBinaryMissing {
				return nil, exec.ErrNotFound
			}

			return cmd, nil
		},
	}
	s.pm.HealthChecker = &MockHealthChecker{}
}

func (s *TestSuite) TearDownSuite(c *C) {
	close(s.shutdownCh)
}

func (s *TestSuite) TestCRUD(c *C) {
	count := 100
	wg := &sync.WaitGroup{}
	pw := &ProcessWatcher{}
	for i := 0; i < count; i++ {
		wg.Add(1)
		go func(i int) {
			defer wg.Done()
			name := "test_crud_process-" + strconv.Itoa(i)
			go func() {
				err := s.pm.ProcessWatch(nil, pw)
				c.Assert(err, IsNil)
			}()

			createReq := &rpc.ProcessCreateRequest{
				Spec: createProcessSpec(name, TestBinary),
			}
			createResp, err := s.pm.ProcessCreate(context.TODO(), createReq)
			c.Assert(err, IsNil)
			c.Assert(createResp.Status.State, Not(Equals), types.ProcessStateStopping)
			c.Assert(createResp.Status.State, Not(Equals), types.ProcessStateStopped)
			c.Assert(createResp.Status.State, Not(Equals), types.ProcessStateError)

			getResp, err := s.pm.ProcessGet(context.TODO(), &rpc.ProcessGetRequest{
				Name: name,
			})
			c.Assert(err, IsNil)
			c.Assert(getResp.Spec.Name, Equals, name)
			c.Assert(getResp.Status.State, Not(Equals), types.ProcessStateStopping)
			c.Assert(getResp.Status.State, Not(Equals), types.ProcessStateStopped)
			c.Assert(getResp.Status.State, Not(Equals), types.ProcessStateError)

			listResp, err := s.pm.ProcessList(context.TODO(), &rpc.ProcessListRequest{})
			c.Assert(err, IsNil)
			c.Assert(listResp.Processes[name], NotNil)
			c.Assert(listResp.Processes[name].Spec.Name, Equals, name)
			c.Assert(listResp.Processes[name].Status.State, Not(Equals), types.ProcessStateStopping)
			c.Assert(listResp.Processes[name].Status.State, Not(Equals), types.ProcessStateStopped)
			c.Assert(listResp.Processes[name].Status.State, Not(Equals), types.ProcessStateError)

			running := false
			for j := 0; j < RetryCount; j++ {
				getResp, err := s.pm.ProcessGet(context.TODO(), &rpc.ProcessGetRequest{
					Name: name,
				})
				c.Assert(err, IsNil)
				if getResp.Status.State == types.ProcessStateRunning {
					running = true
					break
				}
				time.Sleep(RetryInterval)
			}
			c.Assert(running, Equals, true)

			deleteReq := &rpc.ProcessDeleteRequest{
				Name: name,
			}
			deleteResp, err := s.pm.ProcessDelete(context.TODO(), deleteReq)
			c.Assert(err, IsNil)
			c.Assert(deleteResp.Deleted, Equals, true)
			c.Assert(deleteResp.Status.State, Not(Equals), types.ProcessStateStarting)
			c.Assert(deleteResp.Status.State, Not(Equals), types.ProcessStateRunning)
			c.Assert(deleteResp.Status.State, Not(Equals), types.ProcessStateError)
		}(i)
	}
	wg.Wait()
}

// https://github.com/longhorn/longhorn/issues/1113
func (s *TestSuite) TestProcessDeletion(c *C) {
	count := 100
	wg := &sync.WaitGroup{}
	pw := &ProcessWatcher{}
	for i := 0; i < count; i++ {
		wg.Add(1)
		go func(i int) {
			defer wg.Done()
			go func() {
				err := s.pm.ProcessWatch(nil, pw)
				c.Assert(err, IsNil)
			}()
			name := "test_process_deletion-" + strconv.Itoa(i)

			assertProcessCreation(c, s.pm, name, TestBinary)
			assertProcessDeletion(c, s.pm, name)

			deleted, err := waitForProcessListState(s.pm, func(processes map[string]*rpc.ProcessResponse) bool {
				_, exists := processes[name]
				return !exists
			})
			c.Assert(err, IsNil)
			c.Assert(deleted, Equals, true)

			// after previous command delete the process of the same name, creating the process again
			// and make sure it does run and exist for deletion later
			assertProcessCreation(c, s.pm, name, TestBinary)
			assertProcessDeletion(c, s.pm, name)
		}(i)
	}
	wg.Wait()
}

// there was a deadlock when the im.monitor is processing an element
// from the updateChannel, it will try to RLock, to evaluate the existing
// processes. This will deadlock, if during that time a process is
// being replaced, since as part of the replacement. The old process
// when being stopped will try to sent the updated process on the update channel
// while the other scope has acquired a WriteLock.
// Since the IM is waiting on the ReadLock acquisition, while inside of the channel receive
// this will be a total deadlock, since the channel receive can never finish therefore all
// additional sents will be blocked.
// https://github.com/longhorn/longhorn/issues/2697
func (s *TestSuite) TestProcessReplace(c *C) {
	count := 100
	wg := &sync.WaitGroup{}
	pw := &ProcessWatcher{}
	for i := 0; i < count; i++ {
		wg.Add(1)
		go func(i int) {
			defer wg.Done()
			go func() {
				err := s.pm.ProcessWatch(nil, pw)
				c.Assert(err, IsNil)
			}()
			name := "test_process_replace-" + strconv.Itoa(i)
			assertProcessCreation(c, s.pm, name, TestBinary)
			assertProcessReplace(c, s.pm, name, TestBinaryReplace)

			assertProcessDeletion(c, s.pm, name)
			deleted, err := waitForProcessListState(s.pm, func(processes map[string]*rpc.ProcessResponse) bool {
				_, exists := processes[name]
				return !exists
			})
			c.Assert(err, IsNil)
			c.Assert(deleted, Equals, true)
		}(i)
	}
	wg.Wait()
}

// there was a race in the process UpdateChannel assignment
// this lead to a deadlock when the binary couldn't be located.
// https://github.com/longhorn/longhorn/issues/2697
func (s *TestSuite) TestProcessReplaceMissingBinary(c *C) {
	count := 100
	wg := &sync.WaitGroup{}
	pw := &ProcessWatcher{}
	for i := 0; i < count; i++ {
		wg.Add(1)
		go func(i int) {
			defer wg.Done()
			go func() {
				err := s.pm.ProcessWatch(nil, pw)
				c.Assert(err, NotNil)
			}()
			name := "test_process_missing_binary_replace-" + strconv.Itoa(i)
			assertProcessCreation(c, s.pm, name, TestBinary)

			// replacement for a missing binary should error
			_, err := s.pm.ProcessReplace(context.TODO(), &rpc.ProcessReplaceRequest{
				Spec:            createProcessSpec(name, TestBinaryMissing),
				TerminateSignal: "SIGHUP",
			})
			c.Assert(err, NotNil)
		}(i)
	}
	wg.Wait()
}

// there was a nil pointer case, while updating a process that is being
// deleted, since when initially checked the process was still in the map
// but by the time new process has started the old process had been removed
func (s *TestSuite) TestProcessReplaceDuringDeletion(c *C) {
	count := 100
	wg := &sync.WaitGroup{}
	pw := &ProcessWatcher{}
	for i := 0; i < count; i++ {
		wg.Add(1)
		go func(i int) {
			defer wg.Done()
			go func() {
				err := s.pm.ProcessWatch(nil, pw)
				c.Assert(err, NotNil)
			}()
			name := "test_process_deletion_replace-" + strconv.Itoa(i)
			assertProcessCreation(c, s.pm, name, TestBinary)
			assertProcessDeletion(c, s.pm, name)

			// TODO: we should change the deletion handling in a future version
			// since deletion happens async this might error or not
			// depending on if the process has already been deleted from the map
			replaceResp, err := s.pm.ProcessReplace(context.TODO(), &rpc.ProcessReplaceRequest{
				Spec:            createProcessSpec(name, TestBinaryReplace),
				TerminateSignal: "SIGHUP",
			})
			if err != nil {
				c.Assert(err, NotNil)
				c.Assert(status.Code(err), Equals, codes.NotFound)
				return
			}

			c.Assert(err, IsNil)
			c.Assert(replaceResp, NotNil)

			// wait for the replacement process to enter running
			running, err := waitForProcessState(s.pm, name, func(process *rpc.ProcessResponse) bool {
				return process.Status.State == types.ProcessStateRunning
			})
			c.Assert(err, IsNil)
			c.Assert(running, Equals, true)

			// wait for the replacement process cleanup
			assertProcessDeletion(c, s.pm, name)
			deleted, err := waitForProcessListState(s.pm, func(processes map[string]*rpc.ProcessResponse) bool {
				_, exists := processes[name]
				return !exists
			})
			c.Assert(err, IsNil)
			c.Assert(deleted, Equals, true)
		}(i)
	}
	wg.Wait()
}

func assertProcessReplace(c *C, pm *Manager, name, binary string) {
	replaceReq := &rpc.ProcessReplaceRequest{
		Spec:            createProcessSpec(name, binary),
		TerminateSignal: "SIGHUP",
	}
	rsp, err := pm.ProcessReplace(context.TODO(), replaceReq)

	c.Assert(err, IsNil)
	c.Assert(rsp, NotNil)
}

func assertProcessCreation(c *C, pm *Manager, name, binary string) {
	createReq := &rpc.ProcessCreateRequest{
		Spec: createProcessSpec(name, binary),
	}

	createResp, err := pm.ProcessCreate(context.TODO(), createReq)
	c.Assert(err, IsNil)
	c.Assert(createResp.Status.State, Not(Equals), types.ProcessStateStopping)
	c.Assert(createResp.Status.State, Not(Equals), types.ProcessStateStopped)
	c.Assert(createResp.Status.State, Not(Equals), types.ProcessStateError)

	createResp, err = pm.ProcessCreate(context.TODO(), createReq)
	c.Assert(createResp, IsNil)
	c.Assert(err, NotNil)
	c.Assert(status.Code(err), Equals, codes.AlreadyExists)

	running, err := waitForProcessState(pm, name, func(process *rpc.ProcessResponse) bool {
		return process.Status.State == types.ProcessStateRunning
	})
	c.Assert(err, IsNil)
	c.Assert(running, Equals, true)
}

func assertProcessDeletion(c *C, pm *Manager, name string) {
	count := 2
	for j := 0; j < count; j++ {
		deleteReq := &rpc.ProcessDeleteRequest{
			Name: name,
		}
		deleteResp, err := pm.ProcessDelete(context.TODO(), deleteReq)
		if err == nil {
			c.Assert(deleteResp.Deleted, Equals, true)
		} else {
			c.Assert(status.Code(err), Equals, codes.NotFound)
		}
	}
}

func createProcessSpec(name, binary string) *rpc.ProcessSpec {
	return &rpc.ProcessSpec{
		Name:      name,
		Binary:    binary,
		Args:      []string{},
		PortCount: 1,
		PortArgs:  nil,
	}
}

func waitForProcessState(pm *Manager, name string, predicate func(process *rpc.ProcessResponse) bool) (bool, error) {
	for j := 0; j < RetryCount; j++ {
		getResp, err := pm.ProcessGet(context.TODO(), &rpc.ProcessGetRequest{
			Name: name,
		})

		if err != nil {
			return false, err
		}
		if predicate(getResp) {
			return true, nil
		}
		time.Sleep(RetryInterval)
	}

	return false, nil
}

func waitForProcessListState(pm *Manager, predicate func(processes map[string]*rpc.ProcessResponse) bool) (bool, error) {
	// TODO: after a delete operation, it's kinda unexpected one would expect that the process is either gone from the response list
	// 	or that it has some marker that it's in the process of deletion unfortunately the deletion marker is only on the rpc response
	// 	and not the process struct so there is no way for the process list to signal deletion
	for j := 0; j < RetryCount; j++ {
		listResp, err := pm.ProcessList(context.TODO(), &rpc.ProcessListRequest{})
		if err != nil {
			return false, err
		}
		if predicate(listResp.Processes) {
			return true, nil
		}
		time.Sleep(RetryInterval)
	}
	return false, nil
}
</file>

<file path="pkg/process/process.go">
package process

import (
	"sync"
	"syscall"
	"time"

	rpc "github.com/longhorn/types/pkg/generated/imrpc"
	"github.com/sirupsen/logrus"

	"github.com/longhorn/longhorn-instance-manager/pkg/types"
	"github.com/longhorn/longhorn-instance-manager/pkg/util"
)

type State string

const (
	StateStarting = State(types.ProcessStateStarting)
	StateRunning  = State(types.ProcessStateRunning)
	StateStopping = State(types.ProcessStateStopping)
	StateStopped  = State(types.ProcessStateStopped)
	StateError    = State(types.ProcessStateError)
)

type Process struct {
	Name      string
	Binary    string
	Args      []string
	PortCount int32
	PortArgs  []string

	UUID              string
	State             State
	ErrorMsg          string
	Conditions        map[string]bool
	PortStart         int32
	PortEnd           int32
	DeletionTimestamp *time.Time

	lock     *sync.RWMutex
	cmd      Command
	UpdateCh chan *Process

	logger *util.LonghornWriter

	executor      Executor
	healthChecker HealthChecker
}

func (p *Process) Start() error {
	p.lock.Lock()
	defer p.lock.Unlock()

	cmd, err := p.executor.NewCommand(p.Binary, p.Args...)
	if err != nil {
		p.State = StateError
		p.ErrorMsg = err.Error()
		return err
	}
	cmd.SetOutput(p.logger)
	p.cmd = cmd

	probeStopCh := make(chan struct{})
	go func() {
		if err := cmd.Run(); err != nil {
			close(probeStopCh)
			p.lock.Lock()
			p.State = StateError
			p.ErrorMsg = err.Error()
			logrus.Infof("Process Manager: process %v error out, error msg: %v", p.Name, p.ErrorMsg)
			p.lock.Unlock()

			p.UpdateCh <- p
			return
		}
		close(probeStopCh)
		p.lock.Lock()
		p.State = StateStopped
		logrus.Infof("Process Manager: process %v stopped", p.Name)
		p.lock.Unlock()

		p.UpdateCh <- p
	}()

	go func() {
		if p.PortStart != 0 {
			address := util.GetURL("localhost", int(p.PortStart))
			if p.healthChecker.WaitForRunning(address, p.Name, probeStopCh) {
				p.lock.Lock()
				p.State = StateRunning
				p.lock.Unlock()
				p.UpdateCh <- p
				return
			}
			// fail to start the process, then try to stop it.
			if !p.IsStopped() {
				p.Stop()
			}
		} else {
			// Process Manager doesn't know the grpc address. directly set running state
			p.lock.Lock()
			p.State = StateRunning
			p.lock.Unlock()
			p.UpdateCh <- p
		}
	}()

	return nil
}

func (p *Process) RPCResponse() *rpc.ProcessResponse {
	p.lock.RLock()
	defer p.lock.RUnlock()
	if p.ErrorMsg != "" {
		logrus.Warnf("Process update: %v: state %v: errorMsg: %v", p.Name, p.State, p.ErrorMsg)
	}
	return &rpc.ProcessResponse{
		Spec: &rpc.ProcessSpec{
			Name:      p.Name,
			Binary:    p.Binary,
			Args:      p.Args,
			PortCount: p.PortCount,
			PortArgs:  p.PortArgs,
		},

		Status: &rpc.ProcessStatus{
			State:      string(p.State),
			ErrorMsg:   p.ErrorMsg,
			PortStart:  p.PortStart,
			PortEnd:    p.PortEnd,
			Conditions: p.Conditions,
			Uuid:       p.UUID,
		},
	}
}

func (p *Process) Stop() {
	p.StopWithSignal(syscall.SIGINT)
}

func (p *Process) StopWithSignal(signal syscall.Signal) {
	needStop, needTimeoutKill := false, false
	now := time.Now()

	p.lock.Lock()
	if p.State != StateStopping && p.State != StateStopped && p.State != StateError {
		p.State = StateStopping
		p.DeletionTimestamp = &now
		needStop = true
	}
	// Retry the deletion if the process is not stopped in 60 seconds
	if p.DeletionTimestamp != nil && !needStop {
		deleteTimeout := time.Duration(int64(types.WaitInterval) * int64(types.WaitCount))
		if p.DeletionTimestamp.Add(deleteTimeout).Before(now) {
			logrus.Infof("Process Manager: process %v deletion takes more than %vs, will retry it", p.Name, deleteTimeout.Seconds())
			needTimeoutKill = true
		}
	}
	p.lock.Unlock()

	if !needStop && !needTimeoutKill {
		return
	}
	p.UpdateCh <- p

	p.lock.RLock()
	cmd := p.cmd
	p.lock.RUnlock()

	go func() {
		defer func() {
			if err := p.logger.Close(); err != nil {
				logrus.WithError(err).Warnf("Process Manager: failed to close process %v logger", p.Name)
			}
		}()

		if cmd == nil || !cmd.IsRunning() {
			logrus.Errorf("Process Manager: cmd of %v is not running anymore, no need to stop", p.Name)
			if p.State != StateStopped && p.State != StateError {
				p.State = StateStopped
			}
			return
		}

		// no need for lock
		if needStop {
			logrus.Infof("Process Manager: trying to stop process %v", p.Name)
			cmd.StopWithSignal(signal)
			for i := 0; i < types.WaitCount; i++ {
				if p.IsStopped() {
					return
				}
				logrus.Infof("Wait for process %v to shutdown", p.Name)
				time.Sleep(types.WaitInterval)
			}
			logrus.Warnf("Process Manager: cannot graceful stop process %v in %v, will kill the process", p.Name, time.Duration(types.WaitCount)*types.WaitInterval)
		} else if needTimeoutKill {
			logrus.Warnf("Process Manager: somehow timeout stopping process %v, will retry killing the process", p.Name)
		}
		cmd.Kill()
	}()
}

func (p *Process) IsStopped() bool {
	p.lock.RLock()
	defer p.lock.RUnlock()
	return p.State == StateStopped || p.State == StateError
}
</file>

<file path="pkg/process/version.go">
package process

import (
	"context"

	rpc "github.com/longhorn/types/pkg/generated/imrpc"

	"google.golang.org/protobuf/types/known/emptypb"

	"github.com/longhorn/longhorn-instance-manager/pkg/meta"
)

func (pm *Manager) VersionGet(ctx context.Context, empty *emptypb.Empty) (*rpc.VersionResponse, error) {
	v := meta.GetVersion()
	return &rpc.VersionResponse{
		Version:   v.Version,
		GitCommit: v.GitCommit,
		BuildDate: v.BuildDate,

		InstanceManagerAPIVersion:    int64(v.InstanceManagerAPIVersion),
		InstanceManagerAPIMinVersion: int64(v.InstanceManagerAPIMinVersion),

		InstanceManagerProxyAPIVersion:    int64(v.InstanceManagerProxyAPIVersion),
		InstanceManagerProxyAPIMinVersion: int64(v.InstanceManagerProxyAPIMinVersion),
	}, nil
}
</file>

<file path="pkg/proxy/backing_image.go">
package proxy

import (
	"context"
	"fmt"
	"time"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"golang.org/x/sync/errgroup"
	"google.golang.org/protobuf/types/known/emptypb"

	spdkapi "github.com/longhorn/longhorn-spdk-engine/pkg/api"
	spdkclient "github.com/longhorn/longhorn-spdk-engine/pkg/client"
	rpc "github.com/longhorn/types/pkg/generated/imrpc"
)

const (
	maxMonitorRetryCount         = 10
	monitorRetryPollInterval     = 1 * time.Second
	spdkTgtReadinessProbeTimeout = 60 * time.Second
)

func (p *Proxy) SPDKBackingImageCreate(ctx context.Context, req *rpc.SPDKBackingImageCreateRequest) (*rpc.SPDKBackingImageResponse, error) {
	log := logrus.WithFields(logrus.Fields{
		"name":             req.Name,
		"backingImageUUID": req.BackingImageUuid,
		"diskUuid":         req.DiskUuid,
		"size":             req.Size,
		"Checksum":         req.Checksum,
		"FromAddress":      req.FromAddress,
		"SrcLvsUuid":       req.SrcLvsUuid,
	})

	log.Info("Backing Image Server: Creating SPDk Backing Image")
	ret, err := p.spdkLocalClient.BackingImageCreate(req.Name, req.BackingImageUuid, req.DiskUuid, req.Size, req.Checksum, req.FromAddress, req.SrcLvsUuid)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, err.Error())
	}
	return spdkBackingImageToBackingImageResponse(ret), nil
}

func (p *Proxy) SPDKBackingImageDelete(ctx context.Context, req *rpc.SPDKBackingImageDeleteRequest) (*emptypb.Empty, error) {
	log := logrus.WithFields(logrus.Fields{
		"name":     req.Name,
		"diskUuid": req.DiskUuid,
	})
	log.Info("Backing Image Server: Deleting SPDk Backing Image")
	return &emptypb.Empty{}, p.spdkLocalClient.BackingImageDelete(req.Name, req.DiskUuid)
}

func (p *Proxy) SPDKBackingImageGet(ctx context.Context, req *rpc.SPDKBackingImageGetRequest) (*rpc.SPDKBackingImageResponse, error) {
	log := logrus.WithFields(logrus.Fields{
		"name":     req.Name,
		"diskUuid": req.DiskUuid,
	})
	log.Debug("Backing Image Server: Get SPDk Backing Image")
	ret, err := p.spdkLocalClient.BackingImageGet(req.Name, req.DiskUuid)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, err.Error())
	}
	return spdkBackingImageToBackingImageResponse(ret), nil
}

func (p *Proxy) SPDKBackingImageList(ctx context.Context, req *emptypb.Empty) (*rpc.SPDKBackingImageListResponse, error) {
	logrus.Debug("Backing Image Server: List SPDk Backing Image")

	backingImages := map[string]*rpc.SPDKBackingImageResponse{}

	ret, err := p.spdkLocalClient.BackingImageList()
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, err.Error())
	}
	for name, bi := range ret {
		// name is in the form of "bi-%s-disk-%s" because one instance-manager manages multiple disks
		backingImages[name] = spdkBackingImageToBackingImageResponse(bi)
	}
	return &rpc.SPDKBackingImageListResponse{BackingImages: backingImages}, nil
}

func (p *Proxy) SPDKBackingImageWatch(req *emptypb.Empty, srv rpc.ProxyEngineService_SPDKBackingImageWatchServer) error {
	logrus.Info("Start watching SPDK backing image")

	done := make(chan struct{})

	// Create a client for watching SPDK backing image
	spdkClient, err := spdkclient.NewSPDKClient(p.spdkServiceAddress)
	go func() {
		<-done
		logrus.Info("Stopped clients for watching SPDK backing image")
		if spdkClient != nil {
			if closeErr := spdkClient.Close(); closeErr != nil {
				logrus.WithError(closeErr).Warn("Failed to close SPDK client")
			}
		}
		close(done)
	}()
	if err != nil {
		done <- struct{}{}
		return grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create SPDK client").Error())
	}

	notifyChan := make(chan struct{}, 1024)
	defer close(notifyChan)

	g, ctx := errgroup.WithContext(p.ctx)

	g.Go(func() error {
		defer func() {
			// Close the clients for closing streams and unblocking notifier Recv() with error.
			done <- struct{}{}
		}()
		err := p.handleNotify(ctx, notifyChan, srv)
		if err != nil {
			logrus.WithError(err).Error("Failed to handle notify")
		}
		return err
	})

	g.Go(func() error {
		return p.watchSPDKBackingImage(ctx, req, spdkClient, notifyChan)
	})

	if err := g.Wait(); err != nil {
		logrus.WithError(err).Error("Failed to watch backing images")
		return errors.Wrap(err, "failed to watch backing images")
	}

	return nil
}

func (p *Proxy) handleNotify(ctx context.Context, notifyChan chan struct{}, srv rpc.ProxyEngineService_SPDKBackingImageWatchServer) error {
	logrus.Info("Start handling notify")

	for {
		select {
		case <-ctx.Done():
			logrus.Info("Stopped handling notify due to the context done")
			return ctx.Err()
		case <-notifyChan:
			if err := srv.Send(&emptypb.Empty{}); err != nil {
				return errors.Wrap(err, "failed to send backing image response")
			}
		}
	}
}

func (p *Proxy) watchSPDKBackingImage(ctx context.Context, req *emptypb.Empty, client *spdkclient.SPDKClient, notifyChan chan struct{}) error {
	logrus.Info("Start watching SPDK replicas")

	notifier, err := client.BackingImageWatch(ctx)
	if err != nil {
		return errors.Wrap(err, "failed to create SPDK replica watch notifier")
	}

	failureCount := 0
	for {
		if failureCount >= maxMonitorRetryCount {
			logrus.Errorf("Continuously receiving errors for %v times, stopping watching SPDK backing images", maxMonitorRetryCount)
			return fmt.Errorf("continuously receiving errors for %v times, stopping watching SPDK backing images", maxMonitorRetryCount)
		}

		select {
		case <-ctx.Done():
			logrus.Info("Stopped watching SPDK backing images")
			return ctx.Err()
		default:
			_, err := notifier.Recv()
			if err != nil {
				status, ok := grpcstatus.FromError(err)
				if ok && status.Code() == grpccodes.Canceled {
					logrus.WithError(err).Warn("SPDK backing image watch is canceled")
					return err
				}
				logrus.WithError(err).Error("Failed to receive next item in SPDK backing image watch")
				time.Sleep(monitorRetryPollInterval)
				failureCount++
			} else {
				notifyChan <- struct{}{}
			}
		}
	}
}

func spdkBackingImageToBackingImageResponse(bi *spdkapi.BackingImage) *rpc.SPDKBackingImageResponse {
	return &rpc.SPDKBackingImageResponse{
		Spec: &rpc.SPDKBackingImageSpec{
			Name:             bi.Name,
			BackingImageUuid: bi.BackingImageUUID,
			DiskUuid:         bi.LvsUUID,
			Size:             bi.Size,
			Checksum:         bi.ExpectedChecksum,
		},
		Status: &rpc.SPDKBackingImageStatus{
			Progress: bi.Progress,
			State:    bi.State,
			Checksum: bi.CurrentChecksum,
			ErrorMsg: bi.ErrorMsg,
		},
	}
}
</file>

<file path="pkg/proxy/backup.go">
package proxy

import (
	"context"
	"encoding/json"
	"fmt"
	"os"
	"strings"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"google.golang.org/protobuf/types/known/emptypb"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	backupstore "github.com/longhorn/backupstore"
	butil "github.com/longhorn/backupstore/util"
	eclient "github.com/longhorn/longhorn-engine/pkg/controller/client"
	rclient "github.com/longhorn/longhorn-engine/pkg/replica/client"
	esync "github.com/longhorn/longhorn-engine/pkg/sync"
	etypes "github.com/longhorn/longhorn-engine/pkg/types"

	spdkclient "github.com/longhorn/longhorn-spdk-engine/pkg/client"
	rpc "github.com/longhorn/types/pkg/generated/imrpc"

	"github.com/longhorn/longhorn-instance-manager/pkg/util"
)

func (p *Proxy) CleanupBackupMountPoints(ctx context.Context, req *emptypb.Empty) (resp *emptypb.Empty, err error) {
	if err := backupstore.CleanUpAllMounts(); err != nil {
		return &emptypb.Empty{}, grpcstatus.Errorf(grpccodes.Internal, "failed to unmount all mount points: %v", err)
	}
	return &emptypb.Empty{}, nil
}

func (p *Proxy) SnapshotBackup(ctx context.Context, req *rpc.EngineSnapshotBackupRequest) (resp *rpc.EngineSnapshotBackupProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})
	log.Infof("Backing up snapshot %v to backup %v", req.SnapshotName, req.BackupName)

	if err := setEnv(req.Envs); err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to set envs").Error())
	}

	credential, err := butil.GetBackupCredential(req.BackupTarget)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "failed to get backup credential: %v", err)
	}

	labels := getLabels(req.Labels)

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.SnapshotBackup(ctx, req, credential, labels)
}

func (ops V1DataEngineProxyOps) SnapshotBackup(ctx context.Context, req *rpc.EngineSnapshotBackupRequest, credential map[string]string, labels []string) (resp *rpc.EngineSnapshotBackupProxyResponse, err error) {
	task, err := esync.NewTask(ctx, req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName, req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create task").Error())
	}

	recv, err := task.CreateBackup(
		req.BackupName,
		req.SnapshotName,
		req.BackupTarget,
		req.BackingImageName,
		req.BackingImageChecksum,
		req.CompressionMethod,
		int(req.ConcurrentLimit),
		req.StorageClassName,
		labels,
		credential,
		req.Parameters,
	)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create backup").Error())
	}

	return &rpc.EngineSnapshotBackupProxyResponse{
		BackupId:      recv.BackupID,
		Replica:       recv.ReplicaAddress,
		IsIncremental: recv.IsIncremental,
	}, nil
}

func (ops V2DataEngineProxyOps) SnapshotBackup(ctx context.Context, req *rpc.EngineSnapshotBackupRequest, credential map[string]string, labels []string) (resp *rpc.EngineSnapshotBackupProxyResponse, err error) {
	c, err := getSPDKClientFromAddress(req.ProxyEngineRequest.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.ProxyEngineRequest.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	snapshotName := req.SnapshotName
	if snapshotName == "" {
		snapshotName = util.UUID()
	}

	recv, err := c.EngineBackupCreate(&spdkclient.BackupCreateRequest{
		BackupName:           req.BackupName,
		SnapshotName:         snapshotName,
		VolumeName:           req.ProxyEngineRequest.VolumeName,
		EngineName:           req.ProxyEngineRequest.EngineName,
		BackupTarget:         req.BackupTarget,
		StorageClassName:     req.StorageClassName,
		BackingImageName:     req.BackingImageName,
		BackingImageChecksum: req.BackingImageChecksum,
		CompressionMethod:    req.CompressionMethod,
		ConcurrentLimit:      req.ConcurrentLimit,
		Labels:               labels,
		Credential:           credential,
	})
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create backup").Error())
	}

	return &rpc.EngineSnapshotBackupProxyResponse{
		BackupId:      recv.Backup,
		Replica:       recv.ReplicaAddress,
		IsIncremental: recv.IsIncremental,
	}, nil
}

func (p *Proxy) SnapshotBackupStatus(ctx context.Context, req *rpc.EngineSnapshotBackupStatusRequest) (resp *rpc.EngineSnapshotBackupStatusProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})
	log.Tracef("Getting %v backup status from replica %v", req.BackupName, req.ReplicaAddress)

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.SnapshotBackupStatus(ctx, req)
}

func (ops V1DataEngineProxyOps) SnapshotBackupStatus(ctx context.Context, req *rpc.EngineSnapshotBackupStatusRequest) (resp *rpc.EngineSnapshotBackupStatusProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})

	c, err := eclient.NewControllerClient(req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			log.WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	replicas, err := ops.ReplicaList(ctx, req.ProxyEngineRequest)
	if err != nil {
		return nil, err
	}

	replicaAddress := req.ReplicaAddress
	if replicaAddress == "" {
		// find a replica which has the corresponding backup
		for _, r := range replicas.ReplicaList.Replicas {
			mode := etypes.GRPCReplicaModeToReplicaMode(r.Mode)
			if mode != etypes.RW {
				continue
			}

			// We don't know the replicaName here since we retrieved address from the engine, which doesn't know it.
			// Pass it anyway, since the default empty string disables validation and we may know it with a future
			// code change.
			cReplica, err := rclient.NewReplicaClient(r.Address.Address, req.ProxyEngineRequest.VolumeName,
				r.Address.InstanceName)
			if err != nil {
				log.WithError(err).Debugf("Failed to create Replica client with %v", r.Address.Address)
				continue
			}

			_, err = esync.FetchBackupStatus(cReplica, req.BackupName, r.Address.Address)
			if closeErr := cReplica.Close(); closeErr != nil {
				log.WithError(closeErr).Warn("Failed to close Replica client")
			}
			if err == nil {
				replicaAddress = r.Address.Address
				break
			}
		}
	}

	if replicaAddress == "" {
		return nil, errors.Errorf("failed to find a replica with backup %s", req.BackupName)
	}

	for _, r := range replicas.ReplicaList.Replicas {
		if r.Address.Address != replicaAddress {
			continue
		}
		mode := etypes.GRPCReplicaModeToReplicaMode(r.Mode)
		if mode != etypes.RW {
			return nil, errors.Errorf("failed to get backup %v status on unknown replica %s", req.BackupName, replicaAddress)
		}
	}

	// We may know replicaName here. If we don't, we pass an empty string, which disables validation.
	cReplica, err := rclient.NewReplicaClient(replicaAddress, req.ProxyEngineRequest.VolumeName, req.ReplicaName)
	if err != nil {
		return nil, errors.Wrapf(err, "failed to create Replica client with %v", replicaAddress)
	}
	defer func() {
		if closeErr := cReplica.Close(); closeErr != nil {
			log.WithError(closeErr).Warn("Failed to close Replica client")
		}
	}()

	status, err := esync.FetchBackupStatus(cReplica, req.BackupName, replicaAddress)
	if err != nil {
		return nil, err
	}

	return &rpc.EngineSnapshotBackupStatusProxyResponse{
		BackupUrl:      status.BackupURL,
		Error:          status.Error,
		Progress:       int32(status.Progress),
		SnapshotName:   status.SnapshotName,
		State:          status.State,
		ReplicaAddress: replicaAddress,
	}, nil
}

func (ops V2DataEngineProxyOps) SnapshotBackupStatus(ctx context.Context, req *rpc.EngineSnapshotBackupStatusRequest) (resp *rpc.EngineSnapshotBackupStatusProxyResponse, err error) {
	c, err := getSPDKClientFromAddress(req.ProxyEngineRequest.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.ProxyEngineRequest.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	status, err := c.EngineBackupStatus(req.BackupName, req.ProxyEngineRequest.EngineName, req.ReplicaAddress)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get backup status: %v", err)
	}

	return &rpc.EngineSnapshotBackupStatusProxyResponse{
		BackupUrl:      status.BackupUrl,
		Error:          status.Error,
		Progress:       int32(status.Progress),
		SnapshotName:   status.SnapshotName,
		State:          status.State,
		ReplicaAddress: status.ReplicaAddress,
	}, nil
}

func (p *Proxy) BackupRestore(ctx context.Context, req *rpc.EngineBackupRestoreRequest) (resp *rpc.EngineBackupRestoreProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})
	log.Infof("Restoring backup %v to %v", req.Url, req.VolumeName)

	if err := setEnv(req.Envs); err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to set envs").Error())
	}

	credential, err := butil.GetBackupCredential(req.Target)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "failed to get backup credential: %v", err)
	}

	resp = &rpc.EngineBackupRestoreProxyResponse{
		TaskError: []byte{},
	}

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	err = ops.BackupRestore(ctx, req, credential)
	if err != nil {
		errInfo, jsonErr := json.Marshal(err)
		if jsonErr != nil {
			log.WithError(jsonErr).Debugf("Cannot marshal err [%v] to json", err)
		}
		// If the error is not `TaskError`, the marshaled result is an empty json string.
		if string(errInfo) != "{}" {
			resp.TaskError = errInfo
		} else {
			resp.TaskError = []byte(err.Error())
		}
	}

	return resp, nil
}

func (ops V1DataEngineProxyOps) BackupRestore(ctx context.Context, req *rpc.EngineBackupRestoreRequest, credential map[string]string) error {
	task, err := esync.NewTask(ctx, req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return err
	}
	return task.RestoreBackup(req.Url, credential, int(req.ConcurrentLimit))
}

func (ops V2DataEngineProxyOps) BackupRestore(ctx context.Context, req *rpc.EngineBackupRestoreRequest, credential map[string]string) error {
	c, err := getSPDKClientFromAddress(req.ProxyEngineRequest.Address)
	if err != nil {
		return grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.ProxyEngineRequest.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	return c.EngineBackupRestore(&spdkclient.BackupRestoreRequest{
		BackupUrl:       req.Url,
		EngineName:      req.ProxyEngineRequest.EngineName,
		Credential:      credential,
		ConcurrentLimit: req.ConcurrentLimit,
	})
}

func (p *Proxy) BackupRestoreStatus(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineBackupRestoreStatusProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.Address,
		"engineName": req.EngineName,
		"volumeName": req.VolumeName,
		"dataEngine": req.DataEngine,
	})
	log.Trace("Getting backup restore status")

	ops, ok := p.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.BackupRestoreStatus(ctx, req)
}

func (ops V1DataEngineProxyOps) BackupRestoreStatus(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineBackupRestoreStatusProxyResponse, err error) {
	task, err := esync.NewTask(ctx, req.Address, req.VolumeName, req.EngineName)
	if err != nil {
		return nil, err
	}

	recv, err := task.RestoreStatus()
	if err != nil {
		return nil, err
	}

	resp = &rpc.EngineBackupRestoreStatusProxyResponse{
		Status: map[string]*rpc.EngineBackupRestoreStatus{},
	}
	for k, v := range recv {
		resp.Status[k] = &rpc.EngineBackupRestoreStatus{
			IsRestoring:            v.IsRestoring,
			LastRestored:           v.LastRestored,
			CurrentRestoringBackup: v.CurrentRestoringBackup,
			Progress:               int32(v.Progress),
			Error:                  v.Error,
			Filename:               v.Filename,
			State:                  v.State,
			BackupUrl:              v.BackupURL,
		}
	}

	return resp, nil
}

func (ops V2DataEngineProxyOps) BackupRestoreStatus(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineBackupRestoreStatusProxyResponse, err error) {
	c, err := getSPDKClientFromAddress(req.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.Address,
				"engineName": req.EngineName,
				"volumeName": req.VolumeName,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	recv, err := c.EngineRestoreStatus(req.EngineName)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get restore status: %v", err)
	}

	resp = &rpc.EngineBackupRestoreStatusProxyResponse{
		Status: map[string]*rpc.EngineBackupRestoreStatus{},
	}
	for address, status := range recv.Status {
		replicaURL := "tcp://" + address
		resp.Status[replicaURL] = &rpc.EngineBackupRestoreStatus{
			IsRestoring:            status.IsRestoring,
			LastRestored:           status.LastRestored,
			CurrentRestoringBackup: status.CurrentRestoringBackup,
			Progress:               int32(status.Progress),
			Error:                  status.Error,
			State:                  status.State,
			BackupUrl:              status.BackupUrl,
			Filename:               status.DestFileName,
		}
	}
	return resp, nil
}

func setEnv(envs []string) error {
	for _, env := range envs {
		part := strings.SplitN(env, "=", 2)
		if len(part) < 2 {
			continue
		}

		if err := os.Setenv(part[0], part[1]); err != nil {
			return err
		}
	}
	return nil
}

func getLabels(labels map[string]string) []string {
	kvs := []string{}
	for k, v := range labels {
		kvs = append(kvs, fmt.Sprintf("%s=%s", k, v))
	}
	return kvs
}
</file>

<file path="pkg/proxy/healthchecker.go">
package proxy

import (
	"time"

	"github.com/sirupsen/logrus"

	"github.com/longhorn/longhorn-instance-manager/pkg/types"
	"github.com/longhorn/longhorn-instance-manager/pkg/util"
)

type HealthChecker interface {
	IsRunning(address string) bool
	WaitForRunning(address, name string, stopCh chan struct{}) bool
}

type GRPCHealthChecker struct{}

func (c *GRPCHealthChecker) IsRunning(address string) bool {
	return util.GRPCServiceReadinessProbe(address)
}

func (c *GRPCHealthChecker) WaitForRunning(address, name string, stopCh chan struct{}) bool {
	ticker := time.NewTicker(types.WaitInterval)
	defer ticker.Stop()

	for i := 0; i < types.WaitCount; i++ {
		select {
		case <-stopCh:
			logrus.Infof("Stop waiting for gRPC service of proxy %v to start at %v", name, address)
			return false

		case <-ticker.C:
			if c.IsRunning(address) {
				logrus.Infof("Proxy %v has started at %v", name, address)
				return true
			}
			logrus.Infof("Wait for gRPC service of proxy %v to start at %v", name, address)
		}
	}

	return false
}
</file>

<file path="pkg/proxy/import_backupstores.go">
package proxy

import (
	// Involve backupstore drivers for registration
	_ "github.com/longhorn/backupstore/azblob"
	_ "github.com/longhorn/backupstore/cifs"
	_ "github.com/longhorn/backupstore/nfs"
	_ "github.com/longhorn/backupstore/s3"
	_ "github.com/longhorn/backupstore/vfs"
)
</file>

<file path="pkg/proxy/metrics.go">
package proxy

import (
	"context"

	"github.com/sirupsen/logrus"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/types/pkg/generated/enginerpc"

	eclient "github.com/longhorn/longhorn-engine/pkg/controller/client"
	rpc "github.com/longhorn/types/pkg/generated/imrpc"
)

func (p *Proxy) MetricsGet(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineMetricsGetProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.Address,
		"volume":     req.VolumeName,
		"instance":   req.EngineName,
		"dataEngine": req.DataEngine,
	})
	log.Trace("Getting metrics")

	ops, ok := p.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.MetricsGet(ctx, req)
}

func (ops V1DataEngineProxyOps) MetricsGet(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineMetricsGetProxyResponse, err error) {
	c, err := eclient.NewControllerClient(req.Address, req.VolumeName, req.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.Address,
				"volume":     req.VolumeName,
				"instance":   req.EngineName,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	metrics, err := c.MetricsGet()
	if err != nil {
		return nil, err
	}

	return &rpc.EngineMetricsGetProxyResponse{
		Metrics: &enginerpc.Metrics{
			ReadThroughput:  metrics.Throughput.Read,
			WriteThroughput: metrics.Throughput.Write,
			ReadLatency:     metrics.TotalLatency.Read,
			WriteLatency:    metrics.TotalLatency.Write,
			ReadIOPS:        metrics.IOPS.Read,
			WriteIOPS:       metrics.IOPS.Write,
		},
	}, nil
}

func (ops V2DataEngineProxyOps) MetricsGet(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineMetricsGetProxyResponse, err error) {
	c, err := getSPDKClientFromAddress(req.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.Address,
				"volume":     req.VolumeName,
				"instance":   req.EngineName,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	metrics, err := c.MetricsGet(req.EngineName)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get engine %v: %v", req.EngineName, err)
	}

	return &rpc.EngineMetricsGetProxyResponse{
		Metrics: &enginerpc.Metrics{
			ReadThroughput:  metrics.ReadThroughput,
			WriteThroughput: metrics.WriteThroughput,
			ReadLatency:     metrics.ReadLatency,
			WriteLatency:    metrics.WriteLatency,
			ReadIOPS:        metrics.ReadIOPS,
			WriteIOPS:       metrics.WriteIOPS,
		},
	}, nil
}
</file>

<file path="pkg/proxy/proxy.go">
package proxy

import (
	"context"
	"net"
	"strconv"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"google.golang.org/protobuf/types/known/emptypb"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/longhorn-instance-manager/pkg/types"

	"github.com/longhorn/types/pkg/generated/enginerpc"

	eclient "github.com/longhorn/longhorn-engine/pkg/controller/client"
	spdkclient "github.com/longhorn/longhorn-spdk-engine/pkg/client"
	rpc "github.com/longhorn/types/pkg/generated/imrpc"
)

type ProxyOps interface {
	VolumeGet(context.Context, *rpc.ProxyEngineRequest) (*rpc.EngineVolumeGetProxyResponse, error)
	VolumeExpand(context.Context, *rpc.EngineVolumeExpandRequest) (*emptypb.Empty, error)
	VolumeFrontendStart(context.Context, *rpc.EngineVolumeFrontendStartRequest) (*emptypb.Empty, error)
	VolumeFrontendShutdown(context.Context, *rpc.ProxyEngineRequest) (*emptypb.Empty, error)
	VolumeUnmapMarkSnapChainRemovedSet(context.Context, *rpc.EngineVolumeUnmapMarkSnapChainRemovedSetRequest) (*emptypb.Empty, error)

	ReplicaAdd(context.Context, *rpc.EngineReplicaAddRequest) (*emptypb.Empty, error)
	ReplicaList(context.Context, *rpc.ProxyEngineRequest) (*rpc.EngineReplicaListProxyResponse, error)
	ReplicaRebuildingStatus(context.Context, *rpc.ProxyEngineRequest) (*rpc.EngineReplicaRebuildStatusProxyResponse, error)
	ReplicaRebuildingQosSet(context.Context, *rpc.EngineReplicaRebuildingQosSetRequest) (*emptypb.Empty, error)
	ReplicaRemove(context.Context, *rpc.EngineReplicaRemoveRequest) (*emptypb.Empty, error)
	ReplicaVerifyRebuild(context.Context, *rpc.EngineReplicaVerifyRebuildRequest) (*emptypb.Empty, error)
	ReplicaModeUpdate(context.Context, *rpc.EngineReplicaModeUpdateRequest) (*emptypb.Empty, error)
	ReplicaRebuildConcurrentSyncLimitSet(context.Context, *rpc.EngineReplicaRebuildConcurrentSyncLimitSetRequest) (*emptypb.Empty, error)
	ReplicaRebuildConcurrentSyncLimitGet(context.Context, *rpc.ProxyEngineRequest) (*rpc.EngineReplicaRebuildConcurrentSyncLimitGetResponse, error)

	VolumeSnapshot(context.Context, *rpc.EngineVolumeSnapshotRequest) (*rpc.EngineVolumeSnapshotProxyResponse, error)
	SnapshotList(context.Context, *rpc.ProxyEngineRequest) (*rpc.EngineSnapshotListProxyResponse, error)
	SnapshotClone(context.Context, *rpc.EngineSnapshotCloneRequest) (*emptypb.Empty, error)
	SnapshotCloneStatus(context.Context, *rpc.ProxyEngineRequest) (*rpc.EngineSnapshotCloneStatusProxyResponse, error)
	SnapshotRevert(context.Context, *rpc.EngineSnapshotRevertRequest) (*emptypb.Empty, error)
	SnapshotPurge(context.Context, *rpc.EngineSnapshotPurgeRequest) (*emptypb.Empty, error)
	SnapshotPurgeStatus(context.Context, *rpc.ProxyEngineRequest) (*rpc.EngineSnapshotPurgeStatusProxyResponse, error)
	SnapshotRemove(context.Context, *rpc.EngineSnapshotRemoveRequest) (*emptypb.Empty, error)
	SnapshotHash(context.Context, *rpc.EngineSnapshotHashRequest) (*emptypb.Empty, error)
	SnapshotHashStatus(context.Context, *rpc.EngineSnapshotHashStatusRequest) (*rpc.EngineSnapshotHashStatusProxyResponse, error)
	VolumeSnapshotMaxCountSet(context.Context, *rpc.EngineVolumeSnapshotMaxCountSetRequest) (*emptypb.Empty, error)
	VolumeSnapshotMaxSizeSet(context.Context, *rpc.EngineVolumeSnapshotMaxSizeSetRequest) (*emptypb.Empty, error)

	SnapshotBackup(context.Context, *rpc.EngineSnapshotBackupRequest, map[string]string, []string) (*rpc.EngineSnapshotBackupProxyResponse, error)
	SnapshotBackupStatus(context.Context, *rpc.EngineSnapshotBackupStatusRequest) (*rpc.EngineSnapshotBackupStatusProxyResponse, error)
	BackupRestore(context.Context, *rpc.EngineBackupRestoreRequest, map[string]string) error
	BackupRestoreStatus(context.Context, *rpc.ProxyEngineRequest) (*rpc.EngineBackupRestoreStatusProxyResponse, error)

	MetricsGet(context.Context, *rpc.ProxyEngineRequest) (*rpc.EngineMetricsGetProxyResponse, error)
}

type V1DataEngineProxyOps struct{}
type V2DataEngineProxyOps struct{}

type Proxy struct {
	rpc.UnimplementedProxyEngineServiceServer
	ctx           context.Context
	logsDir       string
	HealthChecker HealthChecker
	ops           map[rpc.DataEngine]ProxyOps

	spdkServiceAddress string
	spdkLocalClient    *spdkclient.SPDKClient
}

func NewProxy(ctx context.Context, logsDir, diskServiceAddress, spdkServiceAddress string) (*Proxy, error) {

	ops := map[rpc.DataEngine]ProxyOps{
		rpc.DataEngine_DATA_ENGINE_V1: V1DataEngineProxyOps{},
		rpc.DataEngine_DATA_ENGINE_V2: V2DataEngineProxyOps{},
	}

	spdkLocalClient, err := spdkclient.NewSPDKClient(spdkServiceAddress)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to create SPDK client").Error())
	}

	p := &Proxy{
		ctx:           ctx,
		logsDir:       logsDir,
		HealthChecker: &GRPCHealthChecker{},
		ops:           ops,

		spdkServiceAddress: spdkServiceAddress,
		spdkLocalClient:    spdkLocalClient,
	}

	go p.startMonitoring()

	return p, nil
}

func (p *Proxy) startMonitoring() {
	<-p.ctx.Done()
	logrus.Infof("%s: stopped monitoring due to the context done", types.ProxyGRPCService)
}

func (p *Proxy) ServerVersionGet(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineVersionProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{"serviceURL": req.Address})
	log.Trace("Getting server version")

	c, err := eclient.NewControllerClient(req.Address, req.VolumeName, req.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			log.WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	recv, err := c.VersionDetailGet()
	if err != nil {
		return nil, err
	}

	return &rpc.EngineVersionProxyResponse{
		Version: &enginerpc.VersionOutput{
			Version:                 recv.Version,
			GitCommit:               recv.GitCommit,
			BuildDate:               recv.BuildDate,
			CliAPIVersion:           int64(recv.CLIAPIVersion),
			CliAPIMinVersion:        int64(recv.CLIAPIMinVersion),
			ControllerAPIVersion:    int64(recv.ControllerAPIVersion),
			ControllerAPIMinVersion: int64(recv.ControllerAPIMinVersion),
			DataFormatVersion:       int64(recv.DataFormatVersion),
			DataFormatMinVersion:    int64(recv.DataFormatMinVersion),
		},
	}, nil
}

func getSPDKClientFromAddress(address string) (*spdkclient.SPDKClient, error) {
	host, _, err := net.SplitHostPort(address)
	if err != nil {
		return nil, err
	}

	spdkServiceAddress := net.JoinHostPort(host, strconv.Itoa(types.InstanceManagerSpdkServiceDefaultPort))
	if err != nil {
		return nil, err
	}

	return spdkclient.NewSPDKClient(spdkServiceAddress)
}
</file>

<file path="pkg/proxy/replica.go">
package proxy

import (
	"context"
	"strings"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"google.golang.org/protobuf/types/known/emptypb"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/types/pkg/generated/enginerpc"

	eclient "github.com/longhorn/longhorn-engine/pkg/controller/client"
	esync "github.com/longhorn/longhorn-engine/pkg/sync"
	etypes "github.com/longhorn/longhorn-engine/pkg/types"
	spdktypes "github.com/longhorn/longhorn-spdk-engine/pkg/types"

	rpc "github.com/longhorn/types/pkg/generated/imrpc"

	"github.com/longhorn/longhorn-instance-manager/pkg/types"
)

func (p *Proxy) ReplicaAdd(ctx context.Context, req *rpc.EngineReplicaAddRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL":     req.ProxyEngineRequest.Address,
		"engineName":     req.ProxyEngineRequest.EngineName,
		"volumeName":     req.ProxyEngineRequest.VolumeName,
		"replicaName":    req.ReplicaName,
		"replicaAddress": req.ReplicaAddress,
		"restore":        req.Restore,
		"size":           req.Size,
		"currentSize":    req.CurrentSize,
		"fastSync":       req.FastSync,
		"localSync":      req.LocalSync,
	})
	log.Info("Adding replica")

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.ReplicaAdd(ctx, req)
}

func (ops V1DataEngineProxyOps) ReplicaAdd(ctx context.Context, req *rpc.EngineReplicaAddRequest) (resp *emptypb.Empty, err error) {
	task, err := esync.NewTask(ctx, req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}

	if req.Restore {
		if err := task.AddRestoreReplica(req.Size, req.CurrentSize, req.ReplicaAddress, req.ReplicaName); err != nil {
			return nil, err
		}
	} else {
		var localSync *etypes.FileLocalSync
		if req.LocalSync != nil {
			localSync = &etypes.FileLocalSync{
				SourcePath: req.LocalSync.SourcePath,
				TargetPath: req.LocalSync.TargetPath,
			}
		}
		if err := task.AddReplica(req.Size, req.CurrentSize, req.ReplicaAddress, req.ReplicaName,
			int(req.FileSyncHttpClientTimeout), req.FastSync, localSync, req.GrpcTimeoutSeconds); err != nil {
			return nil, err
		}
	}
	return &emptypb.Empty{}, nil
}

func (ops V2DataEngineProxyOps) ReplicaAdd(ctx context.Context, req *rpc.EngineReplicaAddRequest) (resp *emptypb.Empty, err error) {
	c, err := getSPDKClientFromAddress(req.ProxyEngineRequest.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.ProxyEngineRequest.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL":     req.ProxyEngineRequest.Address,
				"engineName":     req.ProxyEngineRequest.EngineName,
				"volumeName":     req.ProxyEngineRequest.VolumeName,
				"replicaName":    req.ReplicaName,
				"replicaAddress": req.ReplicaAddress,
				"restore":        req.Restore,
				"size":           req.Size,
				"currentSize":    req.CurrentSize,
				"fastSync":       req.FastSync,
				"localSync":      req.LocalSync,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	replicaAddress := strings.TrimPrefix(req.ReplicaAddress, "tcp://")

	err = c.EngineReplicaAdd(req.ProxyEngineRequest.EngineName, req.ReplicaName, replicaAddress)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to add replica %v: %v", replicaAddress, err)
	}
	return &emptypb.Empty{}, nil
}

func (p *Proxy) ReplicaList(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineReplicaListProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.Address,
		"engineName": req.EngineName,
		"volumeName": req.VolumeName,
		"dataEngine": req.DataEngine,
	})
	log.Trace("Listing replicas")

	ops, ok := p.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.ReplicaList(ctx, req)
}

func (ops V1DataEngineProxyOps) ReplicaList(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineReplicaListProxyResponse, err error) {
	c, err := eclient.NewControllerClient(req.Address, req.VolumeName, req.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.Address,
				"engineName": req.EngineName,
				"volumeName": req.VolumeName,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	recv, err := c.ReplicaList()
	if err != nil {
		return nil, err
	}

	replicas := []*enginerpc.ControllerReplica{}
	for _, r := range recv {
		replica := &enginerpc.ControllerReplica{
			Address: &enginerpc.ReplicaAddress{
				Address: r.Address,
			},
			Mode: etypes.ReplicaModeToGRPCReplicaMode(r.Mode),
		}
		replicas = append(replicas, replica)
	}

	return &rpc.EngineReplicaListProxyResponse{
		ReplicaList: &enginerpc.ReplicaListReply{
			Replicas: replicas,
		},
	}, nil
}

func replicaModeToGRPCReplicaMode(mode spdktypes.Mode) enginerpc.ReplicaMode {
	switch mode {
	case spdktypes.ModeWO:
		return enginerpc.ReplicaMode_WO
	case spdktypes.ModeRW:
		return enginerpc.ReplicaMode_RW
	case spdktypes.ModeERR:
		return enginerpc.ReplicaMode_ERR
	}
	return enginerpc.ReplicaMode_ERR
}

func (ops V2DataEngineProxyOps) ReplicaList(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineReplicaListProxyResponse, err error) {
	c, err := getSPDKClientFromAddress(req.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.Address,
				"engineName": req.EngineName,
				"volumeName": req.VolumeName,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	recv, err := c.EngineGet(req.EngineName)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to get engine %v", req.EngineName).Error())
	}

	replicas := []*enginerpc.ControllerReplica{}
	for replicaName, mode := range recv.ReplicaModeMap {
		address, ok := recv.ReplicaAddressMap[replicaName]
		if !ok {
			return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get replica address for %v", replicaName)
		}
		replica := &enginerpc.ControllerReplica{
			Address: &enginerpc.ReplicaAddress{
				Address: types.AddTcpPrefixForAddress(address),
			},
			Mode: replicaModeToGRPCReplicaMode(mode),
		}
		replicas = append(replicas, replica)
	}

	return &rpc.EngineReplicaListProxyResponse{
		ReplicaList: &enginerpc.ReplicaListReply{
			Replicas: replicas,
		},
	}, nil
}

func (p *Proxy) ReplicaRebuildingStatus(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineReplicaRebuildStatusProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.Address,
		"engineName": req.EngineName,
		"volumeName": req.VolumeName,
		"dataEngine": req.DataEngine,
	})
	log.Trace("Getting replica rebuilding status")

	ops, ok := p.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.ReplicaRebuildingStatus(ctx, req)
}

func (ops V1DataEngineProxyOps) ReplicaRebuildingStatus(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineReplicaRebuildStatusProxyResponse, err error) {
	task, err := esync.NewTask(ctx, req.Address, req.VolumeName, req.EngineName)
	if err != nil {
		return nil, err
	}

	recv, err := task.RebuildStatus()
	if err != nil {
		return nil, err
	}

	resp = &rpc.EngineReplicaRebuildStatusProxyResponse{
		Status: make(map[string]*enginerpc.ReplicaRebuildStatusResponse),
	}
	for k, v := range recv {
		resp.Status[k] = &enginerpc.ReplicaRebuildStatusResponse{
			Error:                  v.Error,
			IsRebuilding:           v.IsRebuilding,
			Progress:               int32(v.Progress),
			State:                  v.State,
			FromReplicaAddressList: v.FromReplicaAddressList,
		}
	}

	return resp, nil
}

func (ops V2DataEngineProxyOps) ReplicaRebuildingStatus(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineReplicaRebuildStatusProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.Address,
		"engineName": req.EngineName,
		"volumeName": req.VolumeName,
		"dataEngine": req.DataEngine,
	})

	engineCli, err := getSPDKClientFromAddress(req.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.Address, err)
	}
	defer func() {
		if closeErr := engineCli.Close(); closeErr != nil {
			log.WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	e, err := engineCli.EngineGet(req.EngineName)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get engine %v: %v", req.EngineName, err)
	}

	// TODO: By design, there is one rebuilding replica at most for each volume; hence no need to return a map for rebuilding status.
	resp = &rpc.EngineReplicaRebuildStatusProxyResponse{
		Status: make(map[string]*enginerpc.ReplicaRebuildStatusResponse),
	}
	for replicaName, mode := range e.ReplicaModeMap {
		if mode != spdktypes.ModeWO {
			continue
		}
		replicaAddress := e.ReplicaAddressMap[replicaName]
		if replicaAddress == "" {
			continue
		}
		// TODO: Need to unify the replica address format for v1 and v2 engine
		tcpReplicaAddress := types.AddTcpPrefixForAddress(replicaAddress)
		replicaCli, err := getSPDKClientFromAddress(replicaAddress)
		if err != nil {
			return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from replica address %v: %v", replicaAddress, err)
		}
		defer func() {
			if closeErr := replicaCli.Close(); closeErr != nil {
				log.WithError(closeErr).Warn("Failed to close SPDK client")
			}
		}()

		shallowCopyResp, err := replicaCli.ReplicaRebuildingDstShallowCopyCheck(replicaName)
		if err != nil {
			// Let the upper layer to handle this error rather than considering it as the error message of a rebuilding failure
			return nil, err
		}
		resp.Status[tcpReplicaAddress] = &enginerpc.ReplicaRebuildStatusResponse{
			Error:                  shallowCopyResp.Error,
			IsRebuilding:           shallowCopyResp.TotalState == spdktypes.ProgressStateInProgress,
			Progress:               int32(shallowCopyResp.TotalProgress),
			State:                  shallowCopyResp.TotalState,
			FromReplicaAddressList: []string{types.AddTcpPrefixForAddress(shallowCopyResp.SrcReplicaAddress)},
		}
	}

	return resp, nil
}

func (p *Proxy) ReplicaRebuildingQosSet(ctx context.Context, req *rpc.EngineReplicaRebuildingQosSetRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL":   req.ProxyEngineRequest.Address,
		"engineName":   req.ProxyEngineRequest.EngineName,
		"volumeName":   req.ProxyEngineRequest.VolumeName,
		"dataEngine":   req.ProxyEngineRequest.DataEngine,
		"qosLimitMbps": req.QosLimitMbps,
	})
	log.Trace("Setting qos on replica rebuilding")

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.ReplicaRebuildingQosSet(ctx, req)
}

func (ops V1DataEngineProxyOps) ReplicaRebuildingQosSet(ctx context.Context, req *rpc.EngineReplicaRebuildingQosSetRequest) (resp *emptypb.Empty, err error) {
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
}

func (ops V2DataEngineProxyOps) ReplicaRebuildingQosSet(ctx context.Context, req *rpc.EngineReplicaRebuildingQosSetRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL":   req.ProxyEngineRequest.Address,
		"engineName":   req.ProxyEngineRequest.EngineName,
		"volumeName":   req.ProxyEngineRequest.VolumeName,
		"dataEngine":   req.ProxyEngineRequest.DataEngine,
		"qosLimitMbps": req.QosLimitMbps,
	})

	engineCli, err := getSPDKClientFromAddress(req.ProxyEngineRequest.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.ProxyEngineRequest.Address, err)
	}
	defer func() {
		if closeErr := engineCli.Close(); closeErr != nil {
			log.WithError(closeErr).Warn("Failed to close SPDK engine client")
		}
	}()

	engine, err := engineCli.EngineGet(req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get engine %v: %v", req.ProxyEngineRequest.EngineName, err)
	}

	for replicaName, mode := range engine.ReplicaModeMap {
		if mode != spdktypes.ModeWO {
			continue
		}

		replicaAddress := engine.ReplicaAddressMap[replicaName]
		if replicaAddress == "" {
			log.WithField("replicaName", replicaName).Warn("Empty replica address, skipping QoS set")
			continue
		}

		replicaCli, err := getSPDKClientFromAddress(replicaAddress)
		if err != nil {
			log.WithError(err).WithField("replicaAddress", replicaAddress).
				Warn("Failed to get SPDK client from replica address")
			continue
		}
		defer func() {
			if closeErr := replicaCli.Close(); closeErr != nil {
				log.WithError(closeErr).Warn("Failed to close SPDK replica client")
			}
		}()

		if err := replicaCli.ReplicaRebuildingDstSetQosLimit(replicaName, req.QosLimitMbps); err != nil {
			log.WithError(err).WithFields(logrus.Fields{
				"replicaName":  replicaName,
				"replicaAddr":  replicaAddress,
				"qosLimitMbps": req.QosLimitMbps,
			}).Warn("Failed to set QoS on replica")
			continue
		}

		log.WithFields(logrus.Fields{
			"replicaName":  replicaName,
			"replicaAddr":  replicaAddress,
			"qosLimitMbps": req.QosLimitMbps,
		}).Trace("Successfully set QoS on replica")
	}

	return &emptypb.Empty{}, nil
}

func (p *Proxy) ReplicaVerifyRebuild(ctx context.Context, req *rpc.EngineReplicaVerifyRebuildRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
	})
	log.Infof("Verifying replica %v rebuild", req.ReplicaAddress)

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.ReplicaVerifyRebuild(ctx, req)
}

func (ops V1DataEngineProxyOps) ReplicaVerifyRebuild(ctx context.Context, req *rpc.EngineReplicaVerifyRebuildRequest) (resp *emptypb.Empty, err error) {
	task, err := esync.NewTask(ctx, req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}

	err = task.VerifyRebuildReplica(req.ReplicaAddress, req.ReplicaName)
	if err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (ops V2DataEngineProxyOps) ReplicaVerifyRebuild(ctx context.Context, req *rpc.EngineReplicaVerifyRebuildRequest) (resp *emptypb.Empty, err error) {
	/* TODO: implement this */
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "not implemented")
}

func (p *Proxy) ReplicaRemove(ctx context.Context, req *rpc.EngineReplicaRemoveRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL":     req.ProxyEngineRequest.Address,
		"engineName":     req.ProxyEngineRequest.EngineName,
		"volumeName":     req.ProxyEngineRequest.VolumeName,
		"replicaName":    req.ReplicaName,
		"replicaAddress": req.ReplicaAddress,
	})
	log.Info("Removing replica")

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.ReplicaRemove(ctx, req)
}

func (ops V1DataEngineProxyOps) ReplicaRemove(ctx context.Context, req *rpc.EngineReplicaRemoveRequest) (*emptypb.Empty, error) {
	c, err := eclient.NewControllerClient(req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL":     req.ProxyEngineRequest.Address,
				"engineName":     req.ProxyEngineRequest.EngineName,
				"volumeName":     req.ProxyEngineRequest.VolumeName,
				"replicaName":    req.ReplicaName,
				"replicaAddress": req.ReplicaAddress,
			}).WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	err = ops.cancelSnapshotHashesForReplica(ctx, c, req.ReplicaAddress, req.ProxyEngineRequest)
	if err != nil {
		logrus.WithFields(logrus.Fields{
			"serviceURL":     req.ProxyEngineRequest.Address,
			"engineName":     req.ProxyEngineRequest.EngineName,
			"volumeName":     req.ProxyEngineRequest.VolumeName,
			"replicaName":    req.ReplicaName,
			"replicaAddress": req.ReplicaAddress,
		}).WithError(err).Warn("Failed to stop snapshot hash")
	}

	return nil, c.ReplicaDelete(req.ReplicaAddress)
}

func (ops V1DataEngineProxyOps) cancelSnapshotHashesForReplica(ctx context.Context, c *eclient.ControllerClient, replicaAddress string, proxyEngineRequest *rpc.ProxyEngineRequest) error {
	recv, err := c.ReplicaList()
	if err != nil {
		return err
	}

	snapshotsDiskInfo, err := esync.GetSnapshotsInfo(recv, proxyEngineRequest.VolumeName)
	if err != nil {
		return err
	}
	snapshots := []string{}
	for _, snapshotInfo := range snapshotsDiskInfo {
		snapshots = append(snapshots, snapshotInfo.Name)
	}

	task, err := esync.NewTask(ctx, proxyEngineRequest.Address, proxyEngineRequest.VolumeName, proxyEngineRequest.EngineName)
	if err != nil {
		return err
	}

	if err := task.CancelSnapshotHashJob(replicaAddress, snapshots); err != nil {
		return err
	}

	return nil
}

func (ops V2DataEngineProxyOps) ReplicaRemove(ctx context.Context, req *rpc.EngineReplicaRemoveRequest) (*emptypb.Empty, error) {
	c, err := getSPDKClientFromAddress(req.ProxyEngineRequest.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.ProxyEngineRequest.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL":     req.ProxyEngineRequest.Address,
				"engineName":     req.ProxyEngineRequest.EngineName,
				"volumeName":     req.ProxyEngineRequest.VolumeName,
				"replicaName":    req.ReplicaName,
				"replicaAddress": req.ReplicaAddress,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	replicaAddress := strings.TrimPrefix(req.ReplicaAddress, "tcp://")

	return nil, c.EngineReplicaDelete(req.ProxyEngineRequest.EngineName, req.ReplicaName, replicaAddress)
}

func (p *Proxy) ReplicaModeUpdate(ctx context.Context, req *rpc.EngineReplicaModeUpdateRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
	})
	log.Infof("Updating replica mode to %v", req.Mode)

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}

	return ops.ReplicaModeUpdate(ctx, req)
}

func (ops V1DataEngineProxyOps) ReplicaModeUpdate(ctx context.Context, req *rpc.EngineReplicaModeUpdateRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{"serviceURL": req.ProxyEngineRequest.Address})
	log.Infof("Updating replica mode to %v", req.Mode)

	c, err := eclient.NewControllerClient(req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			log.WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	if _, err = c.ReplicaUpdate(req.ReplicaAddress, etypes.GRPCReplicaModeToReplicaMode(req.Mode)); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (ops V2DataEngineProxyOps) ReplicaModeUpdate(ctx context.Context, req *rpc.EngineReplicaModeUpdateRequest) (resp *emptypb.Empty, err error) {
	/* TODO: implement this */
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "not implemented")
}

func (p *Proxy) ReplicaRebuildConcurrentSyncLimitSet(ctx context.Context, req *rpc.EngineReplicaRebuildConcurrentSyncLimitSetRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
	})
	log.Infof("Updating replica rebuild concurrent sync limit to %d", req.Limit)

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}

	return ops.ReplicaRebuildConcurrentSyncLimitSet(ctx, req)
}

func (ops V1DataEngineProxyOps) ReplicaRebuildConcurrentSyncLimitSet(ctx context.Context, req *rpc.EngineReplicaRebuildConcurrentSyncLimitSetRequest) (resp *emptypb.Empty, err error) {
	c, err := eclient.NewControllerClient(req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}

	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	if err = c.ReplicaRebuildConcurrentSyncLimitSet(int(req.Limit)); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (ops V2DataEngineProxyOps) ReplicaRebuildConcurrentSyncLimitSet(ctx context.Context, req *rpc.EngineReplicaRebuildConcurrentSyncLimitSetRequest) (resp *emptypb.Empty, err error) {
	/* TODO: implement this */
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "not implemented")
}

func (p *Proxy) ReplicaRebuildConcurrentSyncLimitGet(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineReplicaRebuildConcurrentSyncLimitGetResponse, err error) {
	ops, ok := p.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}

	return ops.ReplicaRebuildConcurrentSyncLimitGet(ctx, req)
}

func (ops V1DataEngineProxyOps) ReplicaRebuildConcurrentSyncLimitGet(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineReplicaRebuildConcurrentSyncLimitGetResponse, err error) {
	c, err := eclient.NewControllerClient(req.Address, req.VolumeName,
		req.EngineName)
	if err != nil {
		return nil, err
	}

	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.Address,
				"engineName": req.EngineName,
				"volumeName": req.VolumeName,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	limit, err := c.ReplicaRebuildConcurrentSyncLimitGet()
	if err != nil {
		return nil, err
	}

	return &rpc.EngineReplicaRebuildConcurrentSyncLimitGetResponse{
		Limit: int32(limit),
	}, nil
}

func (ops V2DataEngineProxyOps) ReplicaRebuildConcurrentSyncLimitGet(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineReplicaRebuildConcurrentSyncLimitGetResponse, err error) {
	/* TODO: implement this */
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "not implemented")
}
</file>

<file path="pkg/proxy/snapshot.go">
package proxy

import (
	"context"
	"strconv"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"google.golang.org/protobuf/types/known/emptypb"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/types/pkg/generated/enginerpc"
	"github.com/longhorn/types/pkg/generated/spdkrpc"

	eclient "github.com/longhorn/longhorn-engine/pkg/controller/client"
	esync "github.com/longhorn/longhorn-engine/pkg/sync"
	spdktypes "github.com/longhorn/longhorn-spdk-engine/pkg/types"
	rpc "github.com/longhorn/types/pkg/generated/imrpc"

	"github.com/longhorn/longhorn-instance-manager/pkg/types"
	"github.com/longhorn/longhorn-instance-manager/pkg/util"
)

func (p *Proxy) VolumeSnapshot(ctx context.Context, req *rpc.EngineVolumeSnapshotRequest) (resp *rpc.EngineVolumeSnapshotProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})
	log.Infof("Snapshotting volume: snapshot %v", req.SnapshotVolume.Name)

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.VolumeSnapshot(ctx, req)
}

func (ops V1DataEngineProxyOps) VolumeSnapshot(ctx context.Context, req *rpc.EngineVolumeSnapshotRequest) (resp *rpc.EngineVolumeSnapshotProxyResponse, err error) {
	c, err := eclient.NewControllerClient(req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	recv, err := c.VolumeSnapshot(req.SnapshotVolume.Name, req.SnapshotVolume.Labels, req.SnapshotVolume.FreezeFilesystem)
	if err != nil {
		return nil, err
	}

	return &rpc.EngineVolumeSnapshotProxyResponse{
		Snapshot: &enginerpc.VolumeSnapshotReply{
			Name: recv,
		},
	}, nil
}

func (ops V2DataEngineProxyOps) VolumeSnapshot(ctx context.Context, req *rpc.EngineVolumeSnapshotRequest) (resp *rpc.EngineVolumeSnapshotProxyResponse, err error) {
	c, err := getSPDKClientFromAddress(req.ProxyEngineRequest.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.ProxyEngineRequest.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	snapshotName := req.SnapshotVolume.Name
	if snapshotName == "" {
		snapshotName = util.UUID()
	}

	_, err = c.EngineSnapshotCreate(req.ProxyEngineRequest.EngineName, snapshotName)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to create snapshot %v: %v", snapshotName, err)
	}
	return &rpc.EngineVolumeSnapshotProxyResponse{
		Snapshot: &enginerpc.VolumeSnapshotReply{
			Name: snapshotName,
		},
	}, nil
}

func (p *Proxy) SnapshotList(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineSnapshotListProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.Address,
		"engineName": req.EngineName,
		"volumeName": req.VolumeName,
		"dataEngine": req.DataEngine,
	})
	log.Trace("Listing snapshots")

	ops, ok := p.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.SnapshotList(ctx, req)
}

func (ops V1DataEngineProxyOps) SnapshotList(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineSnapshotListProxyResponse, err error) {
	c, err := eclient.NewControllerClient(req.Address, req.VolumeName, req.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.Address,
				"engineName": req.EngineName,
				"volumeName": req.VolumeName,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	recv, err := c.ReplicaList()
	if err != nil {
		return nil, err
	}

	snapshotsDiskInfo, err := esync.GetSnapshotsInfo(recv, req.VolumeName)
	if err != nil {
		return nil, err
	}

	resp = &rpc.EngineSnapshotListProxyResponse{
		Disks: map[string]*rpc.EngineSnapshotDiskInfo{},
	}
	for k, v := range snapshotsDiskInfo {
		resp.Disks[k] = &rpc.EngineSnapshotDiskInfo{
			Name:        v.Name,
			Parent:      v.Parent,
			Children:    v.Children,
			Removed:     v.Removed,
			UserCreated: v.UserCreated,
			Created:     v.Created,
			Size:        v.Size,
			Labels:      v.Labels,
		}
	}

	return resp, nil
}

func (ops V2DataEngineProxyOps) SnapshotList(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineSnapshotListProxyResponse, err error) {
	c, err := getSPDKClientFromAddress(req.Address)
	if err != nil {
		return nil, errors.Wrapf(err, "failed to get SPDK client from engine address %v", req.Address)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.Address,
				"engineName": req.EngineName,
				"volumeName": req.VolumeName,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	engine, err := c.EngineGet(req.EngineName)
	if err != nil {
		return nil, errors.Wrapf(err, "failed to get engine %v", req.EngineName)
	}
	disks := engine.Snapshots
	if engine.Head != nil {
		disks[engine.Head.Name] = engine.Head
	}

	resp = &rpc.EngineSnapshotListProxyResponse{
		Disks: map[string]*rpc.EngineSnapshotDiskInfo{},
	}
	for snapshotName, snapshot := range disks {
		/*
		 * If the snapshot was created before the introduction of the new attribute SnapshotTimestamp,
		 * and so this one is not available, do fallback over the old one CreationTime.
		 */
		snapshotTime := snapshot.SnapshotTimestamp
		if snapshotTime == "" {
			snapshotTime = snapshot.CreationTime
		}
		resp.Disks[snapshotName] = &rpc.EngineSnapshotDiskInfo{
			Name:        snapshot.Name,
			Parent:      snapshot.Parent,
			Children:    snapshot.Children,
			Removed:     false,
			UserCreated: snapshot.UserCreated,
			Created:     snapshotTime,
			Size:        strconv.FormatUint(snapshot.ActualSize, 10),
			Labels:      map[string]string{},
		}
	}
	return resp, nil
}

func (p *Proxy) SnapshotClone(ctx context.Context, req *rpc.EngineSnapshotCloneRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})
	log.Infof("Cloning snapshot from %v to %v", req.FromEngineAddress, req.ProxyEngineRequest.Address)

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.SnapshotClone(ctx, req)
}

func (ops V1DataEngineProxyOps) SnapshotClone(ctx context.Context, req *rpc.EngineSnapshotCloneRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})

	cFrom, err := eclient.NewControllerClient(req.FromEngineAddress, req.FromVolumeName, req.FromEngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := cFrom.Close(); closeErr != nil {
			log.WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	cTo, err := eclient.NewControllerClient(req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := cTo.Close(); closeErr != nil {
			log.WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	err = esync.CloneSnapshot(cTo, cFrom, req.ProxyEngineRequest.VolumeName, req.FromVolumeName, req.SnapshotName,
		req.ExportBackingImageIfExist, int(req.FileSyncHttpClientTimeout), req.GrpcTimeoutSeconds)
	if err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (ops V2DataEngineProxyOps) SnapshotClone(ctx context.Context, req *rpc.EngineSnapshotCloneRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL":        req.ProxyEngineRequest.Address,
		"engineName":        req.ProxyEngineRequest.EngineName,
		"volumeName":        req.ProxyEngineRequest.VolumeName,
		"dataEngine":        req.ProxyEngineRequest.DataEngine,
		"fromEngineAddress": req.FromEngineAddress,
		"fromEngineName":    req.FromEngineName,
		"fromVolumeName":    req.FromVolumeName,
		"snapshotName":      req.SnapshotName,
	})

	c, err := getSPDKClientFromAddress(req.ProxyEngineRequest.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.ProxyEngineRequest.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			log.WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	err = c.EngineSnapshotClone(req.ProxyEngineRequest.EngineName, req.SnapshotName, req.FromEngineName, req.FromEngineAddress, spdkrpc.CloneMode(req.CloneMode))
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to do clone snapshot %v: %v", req.SnapshotName, err)
	}

	return &emptypb.Empty{}, nil
}

func (p *Proxy) SnapshotCloneStatus(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineSnapshotCloneStatusProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.Address,
		"engineName": req.EngineName,
		"volumeName": req.VolumeName,
		"dataEngine": req.DataEngine,
	})
	log.Trace("Getting snapshot clone status")

	ops, ok := p.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.SnapshotCloneStatus(ctx, req)
}

func (ops V1DataEngineProxyOps) SnapshotCloneStatus(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineSnapshotCloneStatusProxyResponse, err error) {
	c, err := eclient.NewControllerClient(req.Address, req.VolumeName, req.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.Address,
				"engineName": req.EngineName,
				"volumeName": req.VolumeName,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	recv, err := esync.CloneStatus(c, req.VolumeName)
	if err != nil {
		return nil, err
	}

	resp = &rpc.EngineSnapshotCloneStatusProxyResponse{
		Status: map[string]*enginerpc.SnapshotCloneStatusResponse{},
	}
	for k, v := range recv {
		resp.Status[k] = &enginerpc.SnapshotCloneStatusResponse{
			IsCloning:          v.IsCloning,
			Error:              v.Error,
			Progress:           int32(v.Progress),
			State:              v.State,
			FromReplicaAddress: v.FromReplicaAddress,
			SnapshotName:       v.SnapshotName,
		}
	}

	return resp, nil
}

func (ops V2DataEngineProxyOps) SnapshotCloneStatus(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineSnapshotCloneStatusProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.Address,
		"engineName": req.EngineName,
		"volumeName": req.VolumeName,
		"dataEngine": req.DataEngine,
	})
	c, err := getSPDKClientFromAddress(req.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			log.WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	recv, err := c.EngineGet(req.EngineName)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to get engine %v", req.EngineName).Error())
	}

	replicaName, replicaAddress := "", ""
	for rName, mode := range recv.ReplicaModeMap {
		if mode != spdktypes.ModeRW {
			continue
		}
		address, ok := recv.ReplicaAddressMap[rName]
		if !ok {
			return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get replica address for %v", replicaName)
		}
		replicaName = rName
		replicaAddress = address
		break
	}
	if replicaName == "" || replicaAddress == "" {
		return nil, grpcstatus.Error(grpccodes.Internal, "cannot find a RW replica")
	}

	replicaClient, err := getSPDKClientFromAddress(replicaAddress)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "cannot ger client for replica %v", replicaName)
	}
	defer func() {
		if closeErr := replicaClient.Close(); closeErr != nil {
			log.WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	status, err := replicaClient.ReplicaSnapshotCloneDstStatusCheck(replicaName)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get clone status for %v: %v", replicaName, err)
	}
	resp = &rpc.EngineSnapshotCloneStatusProxyResponse{
		Status: map[string]*enginerpc.SnapshotCloneStatusResponse{},
	}
	tcpReplicaAddress := types.AddTcpPrefixForAddress(replicaAddress)
	resp.Status[tcpReplicaAddress] = &enginerpc.SnapshotCloneStatusResponse{
		IsCloning:          status.IsCloning,
		Error:              status.Error,
		Progress:           int32(status.Progress),
		State:              status.State,
		FromReplicaAddress: status.SrcReplicaAddress,
		SnapshotName:       status.SnapshotName,
	}

	return resp, nil
}

func (p *Proxy) SnapshotRevert(ctx context.Context, req *rpc.EngineSnapshotRevertRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})
	log.Infof("Reverting snapshot %v", req.Name)

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.SnapshotRevert(ctx, req)
}

func (ops V1DataEngineProxyOps) SnapshotRevert(ctx context.Context, req *rpc.EngineSnapshotRevertRequest) (resp *emptypb.Empty, err error) {
	c, err := eclient.NewControllerClient(req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	if err := c.VolumeRevert(req.Name); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (ops V2DataEngineProxyOps) SnapshotRevert(ctx context.Context, req *rpc.EngineSnapshotRevertRequest) (resp *emptypb.Empty, err error) {
	c, err := getSPDKClientFromAddress(req.ProxyEngineRequest.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.ProxyEngineRequest.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	err = c.EngineSnapshotRevert(req.ProxyEngineRequest.EngineName, req.Name)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to create snapshot %v: %v", req.Name, err)
	}

	return &emptypb.Empty{}, nil
}

func (p *Proxy) SnapshotPurge(ctx context.Context, req *rpc.EngineSnapshotPurgeRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})
	log.Info("Purging snapshots")

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.SnapshotPurge(ctx, req)
}

func (ops V1DataEngineProxyOps) SnapshotPurge(ctx context.Context, req *rpc.EngineSnapshotPurgeRequest) (resp *emptypb.Empty, err error) {
	task, err := esync.NewTask(ctx, req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}

	if err := task.PurgeSnapshots(req.SkipIfInProgress); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (ops V2DataEngineProxyOps) SnapshotPurge(ctx context.Context, req *rpc.EngineSnapshotPurgeRequest) (resp *emptypb.Empty, err error) {
	c, err := getSPDKClientFromAddress(req.ProxyEngineRequest.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.ProxyEngineRequest.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	// For v2 Data Engine, snapshot purge is no longer a time-consuming operation
	err = c.EngineSnapshotPurge(req.ProxyEngineRequest.EngineName)
	return &emptypb.Empty{}, nil
}

func (p *Proxy) SnapshotPurgeStatus(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineSnapshotPurgeStatusProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.Address,
		"engineName": req.EngineName,
		"volumeName": req.VolumeName,
		"dataEngine": req.DataEngine,
	})
	log.Trace("Getting snapshot purge status")

	ops, ok := p.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.SnapshotPurgeStatus(ctx, req)
}

func (ops V1DataEngineProxyOps) SnapshotPurgeStatus(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineSnapshotPurgeStatusProxyResponse, err error) {
	task, err := esync.NewTask(ctx, req.Address, req.VolumeName, req.EngineName)
	if err != nil {
		return nil, err
	}

	recv, err := task.PurgeSnapshotStatus()
	if err != nil {
		return nil, err
	}

	resp = &rpc.EngineSnapshotPurgeStatusProxyResponse{
		Status: map[string]*enginerpc.SnapshotPurgeStatusResponse{},
	}
	for k, v := range recv {
		resp.Status[k] = &enginerpc.SnapshotPurgeStatusResponse{
			IsPurging: v.IsPurging,
			Error:     v.Error,
			Progress:  int32(v.Progress),
			State:     v.State,
		}
	}

	return resp, nil
}

func (ops V2DataEngineProxyOps) SnapshotPurgeStatus(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineSnapshotPurgeStatusProxyResponse, err error) {
	/* TODO: implement this */
	return &rpc.EngineSnapshotPurgeStatusProxyResponse{
		Status: map[string]*enginerpc.SnapshotPurgeStatusResponse{},
	}, nil
}

func (p *Proxy) SnapshotRemove(ctx context.Context, req *rpc.EngineSnapshotRemoveRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})
	log.Infof("Removing snapshots %v", req.Names)

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.SnapshotRemove(ctx, req)
}

func (ops V1DataEngineProxyOps) SnapshotRemove(ctx context.Context, req *rpc.EngineSnapshotRemoveRequest) (resp *emptypb.Empty, err error) {
	task, err := esync.NewTask(ctx, req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}

	var lastErr error
	for _, name := range req.Names {
		if err := task.DeleteSnapshot(name); err != nil {
			if err != nil {
				lastErr = err
				logrus.WithError(err).Warnf("Failed to delete snapshot %s", name)
			}
		}
	}

	return &emptypb.Empty{}, lastErr
}

func (ops V2DataEngineProxyOps) SnapshotRemove(ctx context.Context, req *rpc.EngineSnapshotRemoveRequest) (resp *emptypb.Empty, err error) {
	c, err := getSPDKClientFromAddress(req.ProxyEngineRequest.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.ProxyEngineRequest.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	var lastErr error
	for _, name := range req.Names {
		err = c.EngineSnapshotDelete(req.ProxyEngineRequest.EngineName, name)
		if err != nil {
			lastErr = err
			logrus.WithError(err).Warnf("Failed to delete snapshot %s", name)
		}
	}

	return &emptypb.Empty{}, lastErr
}

func (p *Proxy) SnapshotHash(ctx context.Context, req *rpc.EngineSnapshotHashRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})
	log.Infof("Hashing snapshot %v with rehash %v", req.SnapshotName, req.Rehash)

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.SnapshotHash(ctx, req)
}

func (ops V1DataEngineProxyOps) SnapshotHash(ctx context.Context, req *rpc.EngineSnapshotHashRequest) (resp *emptypb.Empty, err error) {
	task, err := esync.NewTask(ctx, req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}

	if err := task.HashSnapshot(req.SnapshotName, req.Rehash); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (ops V2DataEngineProxyOps) SnapshotHash(ctx context.Context, req *rpc.EngineSnapshotHashRequest) (resp *emptypb.Empty, err error) {
	c, err := getSPDKClientFromAddress(req.ProxyEngineRequest.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.ProxyEngineRequest.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	err = c.EngineSnapshotHash(req.ProxyEngineRequest.EngineName, req.SnapshotName, req.Rehash)
	return &emptypb.Empty{}, err
}

func (p *Proxy) SnapshotHashStatus(ctx context.Context, req *rpc.EngineSnapshotHashStatusRequest) (resp *rpc.EngineSnapshotHashStatusProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})
	log.Trace("Getting snapshot hash status")

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.SnapshotHashStatus(ctx, req)
}

func (ops V1DataEngineProxyOps) SnapshotHashStatus(ctx context.Context, req *rpc.EngineSnapshotHashStatusRequest) (resp *rpc.EngineSnapshotHashStatusProxyResponse, err error) {
	task, err := esync.NewTask(ctx, req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}

	recv, err := task.HashSnapshotStatus(req.SnapshotName)
	if err != nil {
		return nil, err
	}

	resp = &rpc.EngineSnapshotHashStatusProxyResponse{
		Status: map[string]*enginerpc.SnapshotHashStatusResponse{},
	}
	for k, v := range recv {
		resp.Status[k] = &enginerpc.SnapshotHashStatusResponse{
			State:             v.State,
			Checksum:          v.Checksum,
			Error:             v.Error,
			SilentlyCorrupted: v.SilentlyCorrupted,
		}
	}

	return resp, nil
}

func (ops V2DataEngineProxyOps) SnapshotHashStatus(ctx context.Context, req *rpc.EngineSnapshotHashStatusRequest) (resp *rpc.EngineSnapshotHashStatusProxyResponse, err error) {
	c, err := getSPDKClientFromAddress(req.ProxyEngineRequest.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.ProxyEngineRequest.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	recv, err := c.EngineSnapshotHashStatus(req.ProxyEngineRequest.EngineName, req.SnapshotName)
	if err != nil {
		return nil, err
	}

	resp = &rpc.EngineSnapshotHashStatusProxyResponse{
		Status: map[string]*enginerpc.SnapshotHashStatusResponse{},
	}
	for k, v := range recv.Status {
		resp.Status[k] = &enginerpc.SnapshotHashStatusResponse{
			State:             v.State,
			Checksum:          v.Checksum,
			Error:             v.Error,
			SilentlyCorrupted: v.SilentlyCorrupted,
		}
	}

	return resp, nil
}
</file>

<file path="pkg/proxy/volume.go">
package proxy

import (
	"context"
	"crypto/sha256"
	"encoding/hex"

	"github.com/sirupsen/logrus"
	"google.golang.org/protobuf/types/known/emptypb"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/types/pkg/generated/enginerpc"

	lhns "github.com/longhorn/go-common-libs/ns"
	lhtypes "github.com/longhorn/go-common-libs/types"
	eclient "github.com/longhorn/longhorn-engine/pkg/controller/client"
	rpc "github.com/longhorn/types/pkg/generated/imrpc"

	"github.com/longhorn/longhorn-instance-manager/pkg/util"
)

func (p *Proxy) VolumeGet(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineVolumeGetProxyResponse, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.Address,
		"engineName": req.EngineName,
		"volumeName": req.VolumeName,
		"dataEngine": req.DataEngine,
	})
	log.Trace("Getting volume")

	ops, ok := p.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.VolumeGet(ctx, req)
}

func (ops V1DataEngineProxyOps) VolumeGet(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineVolumeGetProxyResponse, err error) {
	c, err := eclient.NewControllerClient(req.Address, req.VolumeName, req.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.Address,
				"engineName": req.EngineName,
				"volumeName": req.VolumeName,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	recv, err := c.VolumeGet()
	if err != nil {
		return nil, err
	}

	return &rpc.EngineVolumeGetProxyResponse{
		Volume: &enginerpc.Volume{
			Name:                      recv.Name,
			Size:                      recv.Size,
			ReplicaCount:              int32(recv.ReplicaCount),
			Endpoint:                  recv.Endpoint,
			Frontend:                  recv.Frontend,
			FrontendState:             recv.FrontendState,
			IsExpanding:               recv.IsExpanding,
			LastExpansionError:        recv.LastExpansionError,
			LastExpansionFailedAt:     recv.LastExpansionFailedAt,
			UnmapMarkSnapChainRemoved: recv.UnmapMarkSnapChainRemoved,
			SnapshotMaxCount:          int32(recv.SnapshotMaxCount),
			SnapshotMaxSize:           recv.SnapshotMaxSize,
		},
	}, nil
}

func (ops V2DataEngineProxyOps) VolumeGet(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *rpc.EngineVolumeGetProxyResponse, err error) {
	c, err := getSPDKClientFromAddress(req.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.Address,
				"engineName": req.EngineName,
				"volumeName": req.VolumeName,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	recv, err := c.EngineGet(req.EngineName)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get engine %v: %v", req.EngineName, err)
	}

	return &rpc.EngineVolumeGetProxyResponse{
		Volume: &enginerpc.Volume{
			Name:                      recv.Name,
			Size:                      int64(recv.SpecSize),
			ReplicaCount:              int32(len(recv.ReplicaAddressMap)),
			Endpoint:                  recv.Endpoint,
			Frontend:                  recv.Frontend,
			FrontendState:             "",
			IsExpanding:               false,
			LastExpansionError:        "",
			LastExpansionFailedAt:     "",
			UnmapMarkSnapChainRemoved: false,
		},
	}, nil
}

func (p *Proxy) VolumeExpand(ctx context.Context, req *rpc.EngineVolumeExpandRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})
	log.Infof("Expanding volume to size %v", req.Expand.Size)

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.VolumeExpand(ctx, req)
}

func (ops V1DataEngineProxyOps) VolumeExpand(ctx context.Context, req *rpc.EngineVolumeExpandRequest) (resp *emptypb.Empty, err error) {
	c, err := eclient.NewControllerClient(req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	err = c.VolumeExpand(req.Expand.Size)
	if err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (ops V2DataEngineProxyOps) VolumeExpand(ctx context.Context, req *rpc.EngineVolumeExpandRequest) (resp *emptypb.Empty, err error) {
	c, err := getSPDKClientFromAddress(req.ProxyEngineRequest.Address)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client from engine address %v: %v", req.ProxyEngineRequest.Address, err)
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close SPDK client")
		}
	}()

	err = c.EngineExpand(ctx, req.ProxyEngineRequest.EngineName, uint64(req.Expand.Size))
	if err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (p *Proxy) VolumeFrontendStart(ctx context.Context, req *rpc.EngineVolumeFrontendStartRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})
	log.Infof("Starting volume frontend %v", req.FrontendStart.Frontend)

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.VolumeFrontendStart(ctx, req)
}

func (ops V1DataEngineProxyOps) VolumeFrontendStart(ctx context.Context, req *rpc.EngineVolumeFrontendStartRequest) (resp *emptypb.Empty, err error) {
	c, err := eclient.NewControllerClient(req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	err = c.VolumeFrontendStart(req.FrontendStart.Frontend)
	if err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (ops V2DataEngineProxyOps) VolumeFrontendStart(ctx context.Context, req *rpc.EngineVolumeFrontendStartRequest) (resp *emptypb.Empty, err error) {
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "VolumeFrontendStart is not yet implemented for V2 engine")
}

func (p *Proxy) VolumeFrontendShutdown(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.Address,
		"engineName": req.EngineName,
		"volumeName": req.VolumeName,
		"dataEngine": req.DataEngine,
	})
	log.Info("Shutting down volume frontend")

	ops, ok := p.ops[req.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.DataEngine)
	}
	return ops.VolumeFrontendShutdown(ctx, req)
}

func (ops V1DataEngineProxyOps) VolumeFrontendShutdown(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *emptypb.Empty, err error) {
	c, err := eclient.NewControllerClient(req.Address, req.VolumeName, req.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.Address,
				"engineName": req.EngineName,
				"volumeName": req.VolumeName,
				"dataEngine": req.DataEngine,
			}).WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	err = c.VolumeFrontendShutdown()
	if err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (ops V2DataEngineProxyOps) VolumeFrontendShutdown(ctx context.Context, req *rpc.ProxyEngineRequest) (resp *emptypb.Empty, err error) {
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "VolumeFrontendShutdown is not yet implemented for V2 engine")
}

func (p *Proxy) VolumeUnmapMarkSnapChainRemovedSet(ctx context.Context, req *rpc.EngineVolumeUnmapMarkSnapChainRemovedSetRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})
	log.Infof("Setting volume flag UnmapMarkSnapChainRemoved to %v", req.UnmapMarkSnap.Enabled)

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.VolumeUnmapMarkSnapChainRemovedSet(ctx, req)
}

func (ops V1DataEngineProxyOps) VolumeUnmapMarkSnapChainRemovedSet(ctx context.Context, req *rpc.EngineVolumeUnmapMarkSnapChainRemovedSetRequest) (resp *emptypb.Empty, err error) {
	c, err := eclient.NewControllerClient(req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	err = c.VolumeUnmapMarkSnapChainRemovedSet(req.UnmapMarkSnap.Enabled)
	if err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (ops V2DataEngineProxyOps) VolumeUnmapMarkSnapChainRemovedSet(ctx context.Context, req *rpc.EngineVolumeUnmapMarkSnapChainRemovedSetRequest) (resp *emptypb.Empty, err error) {
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "VolumeUnmapMarkSnapChainRemovedSet is not yet implemented for V2 engine")
}

func (p *Proxy) VolumeSnapshotMaxCountSet(ctx context.Context, req *rpc.EngineVolumeSnapshotMaxCountSetRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})
	log.Infof("Setting volume flag SnapshotMaxCount to %v", req.Count.Count)

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.VolumeSnapshotMaxCountSet(ctx, req)
}

func (ops V1DataEngineProxyOps) VolumeSnapshotMaxCountSet(ctx context.Context, req *rpc.EngineVolumeSnapshotMaxCountSetRequest) (resp *emptypb.Empty, err error) {
	c, err := eclient.NewControllerClient(req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	err = c.VolumeSnapshotMaxCountSet(int(req.Count.Count))
	if err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (ops V2DataEngineProxyOps) VolumeSnapshotMaxCountSet(ctx context.Context, req *rpc.EngineVolumeSnapshotMaxCountSetRequest) (resp *emptypb.Empty, err error) {
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "VolumeSnapshotMaxCountSet is not yet implemented for V2 engine")
}

func (p *Proxy) VolumeSnapshotMaxSizeSet(ctx context.Context, req *rpc.EngineVolumeSnapshotMaxSizeSetRequest) (resp *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"serviceURL": req.ProxyEngineRequest.Address,
		"engineName": req.ProxyEngineRequest.EngineName,
		"volumeName": req.ProxyEngineRequest.VolumeName,
		"dataEngine": req.ProxyEngineRequest.DataEngine,
	})
	log.Infof("Setting volume flag SnapshotMaxSize to %v", req.Size.Size)

	ops, ok := p.ops[req.ProxyEngineRequest.DataEngine]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "unsupported data engine %v", req.ProxyEngineRequest.DataEngine)
	}
	return ops.VolumeSnapshotMaxSizeSet(ctx, req)
}

func (ops V1DataEngineProxyOps) VolumeSnapshotMaxSizeSet(ctx context.Context, req *rpc.EngineVolumeSnapshotMaxSizeSetRequest) (resp *emptypb.Empty, err error) {
	c, err := eclient.NewControllerClient(req.ProxyEngineRequest.Address, req.ProxyEngineRequest.VolumeName,
		req.ProxyEngineRequest.EngineName)
	if err != nil {
		return nil, err
	}
	defer func() {
		if closeErr := c.Close(); closeErr != nil {
			logrus.WithFields(logrus.Fields{
				"serviceURL": req.ProxyEngineRequest.Address,
				"engineName": req.ProxyEngineRequest.EngineName,
				"volumeName": req.ProxyEngineRequest.VolumeName,
				"dataEngine": req.ProxyEngineRequest.DataEngine,
			}).WithError(closeErr).Warn("Failed to close Controller client")
		}
	}()

	err = c.VolumeSnapshotMaxSizeSet(req.Size.Size)
	if err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (ops V2DataEngineProxyOps) VolumeSnapshotMaxSizeSet(ctx context.Context, req *rpc.EngineVolumeSnapshotMaxSizeSetRequest) (resp *emptypb.Empty, err error) {
	return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "VolumeSnapshotMaxSizeSet is not yet implemented for V2 engine")
}

func (p *Proxy) RemountReadOnlyVolume(ctx context.Context, req *rpc.RemountVolumeRequest) (resp *emptypb.Empty, err error) {
	volumeName := req.VolumeName
	volumeNameSHA := sha256.Sum256([]byte(volumeName))
	volumeNameSHAStr := hex.EncodeToString(volumeNameSHA[:])

	volumeMountPointMap, err := util.GetVolumeMountPointMap()
	if err != nil {
		return &emptypb.Empty{}, err
	}

	namespaces := []lhtypes.Namespace{lhtypes.NamespaceMnt, lhtypes.NamespaceNet}
	nsexec, err := lhns.NewNamespaceExecutor(lhtypes.ProcessNone, lhtypes.ProcDirectory, namespaces)
	if err != nil {
		return &emptypb.Empty{}, err
	}

	if mp, exists := volumeMountPointMap[volumeNameSHAStr]; exists {
		opts := []string{
			"-o",
			"remount,rw",
			mp.Path,
		}
		if _, err := nsexec.Execute(nil, "mount", opts, lhtypes.ExecuteDefaultTimeout); err != nil {
			return nil, grpcstatus.Errorf(grpccodes.Internal, "remount failed with error: %v", err)
		}
	}

	return &emptypb.Empty{}, nil
}
</file>

<file path="pkg/types/types.go">
package types

import (
	"time"
)

const (
	GRPCServiceTimeout = 3 * time.Minute

	ProcessStateRunning  = "running"
	ProcessStateStarting = "starting"
	ProcessStateStopped  = "stopped"
	ProcessStateStopping = "stopping"
	ProcessStateError    = "error"

	DiskGrpcService           = "disk gRPC server"
	SpdkGrpcService           = "spdk gRPC server"
	ProcessManagerGrpcService = "process-manager gRPC server"
	InstanceGrpcService       = "instance gRPC server"
	ProxyGRPCService          = "proxy gRPC server"
)

const (
	InstanceManagerProcessManagerServiceDefaultPort = 8500
	InstanceManagerProxyServiceDefaultPort          = InstanceManagerProcessManagerServiceDefaultPort + 1 // 8501
	InstanceManagerDiskServiceDefaultPort           = InstanceManagerProcessManagerServiceDefaultPort + 2 // 8502
	InstanceManagerInstanceServiceDefaultPort       = InstanceManagerProcessManagerServiceDefaultPort + 3 // 8503
	InstanceManagerSpdkServiceDefaultPort           = InstanceManagerProcessManagerServiceDefaultPort + 4 // 8504
)

var (
	WaitInterval = 100 * time.Millisecond
	WaitCount    = 600
)

const (
	RetryInterval = 3 * time.Second
	RetryCounts   = 3
)

const (
	InstanceTypeEngine  = "engine"
	InstanceTypeReplica = "replica"
)

const (
	EngineConditionFilesystemReadOnly = "FilesystemReadOnly"
)

const TcpAddressPrefix = "tcp://"

func AddTcpPrefixForAddress(address string) string {
	if address == "" {
		return ""
	}

	return TcpAddressPrefix + address
}
</file>

<file path="pkg/util/broadcaster/broadcaster.go">
package broadcaster

import (
	"context"
	"sync"
)

type ConnectFunc func() (chan interface{}, error)

type Broadcaster struct {
	sync.Mutex
	running bool
	subs    map[chan interface{}]struct{}
}

func (b *Broadcaster) Subscribe(ctx context.Context, connect ConnectFunc) (<-chan interface{}, error) {
	b.Lock()
	defer b.Unlock()

	if !b.running {
		if err := b.start(connect); err != nil {
			return nil, err
		}
	}

	sub := make(chan interface{}, 100)
	if b.subs == nil {
		b.subs = map[chan interface{}]struct{}{}
	}
	b.subs[sub] = struct{}{}
	go func() {
		<-ctx.Done()
		b.unsub(sub, true)
	}()

	return sub, nil
}

func (b *Broadcaster) unsub(sub chan interface{}, lock bool) {
	if lock {
		b.Lock()
	}
	if _, ok := b.subs[sub]; ok {
		close(sub)
		delete(b.subs, sub)
	}
	if lock {
		b.Unlock()
	}
}

func (b *Broadcaster) start(connect ConnectFunc) error {
	c, err := connect()
	if err != nil {
		return err
	}

	go b.stream(c)
	b.running = true
	return nil
}

func (b *Broadcaster) stream(input chan interface{}) {
	for item := range input {
		b.Lock()
		for sub := range b.subs {
			select {
			case sub <- item:
			default:
				// Slow consumer, drop
				go b.unsub(sub, true)
			}
		}
		b.Unlock()
	}

	b.Lock()
	for sub := range b.subs {
		b.unsub(sub, false)
	}
	b.running = false
	b.Unlock()
}
</file>

<file path="pkg/util/grpcutil_test.go">
package util

import (
	"testing"
)

func Test_parseEndpoint(t *testing.T) {
	type args struct {
		ep string
	}
	tests := []struct {
		name        string
		args        args
		wantProto   string
		wantAddress string
		wantErr     bool
	}{
		{name: "testEndpointUnix", args: args{ep: "unix:///tmp/test.sock"}, wantProto: "unix", wantAddress: "/tmp/test.sock", wantErr: false},
		{name: "testEndpointTcp", args: args{ep: "tcp://127.0.0.1:8500"}, wantProto: "tcp", wantAddress: "127.0.0.1:8500", wantErr: false},
		{name: "testEndpointProtoMissingFallback", args: args{ep: "localhost:8500"}, wantProto: "tcp", wantAddress: "localhost:8500", wantErr: false},
		{name: "testEndpointProtoUnsupported", args: args{ep: "unsupported://127.0.0.1:8500"}, wantProto: "", wantAddress: "", wantErr: true},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			gotProto, gotAddress, err := parseEndpoint(tt.args.ep)
			if (err != nil) != tt.wantErr {
				t.Errorf("parseEndpoint() error = %v, wantErr %v", err, tt.wantErr)
				return
			}
			if gotProto != tt.wantProto {
				t.Errorf("parseEndpoint() gotProto = %v, want %v", gotProto, tt.wantProto)
			}
			if gotAddress != tt.wantAddress {
				t.Errorf("parseEndpoint() gotAddress = %v, want %v", gotAddress, tt.wantAddress)
			}
		})
	}
}
</file>

<file path="pkg/util/grpcutil.go">
package util

import (
	"context"
	"crypto/tls"
	"crypto/x509"
	"errors"
	"fmt"
	"net"
	"os"
	"strings"
	"time"

	"google.golang.org/grpc"
	"google.golang.org/grpc/backoff"
	"google.golang.org/grpc/credentials"
	"google.golang.org/grpc/credentials/insecure"
	"google.golang.org/grpc/keepalive"
)

func unixDialer(ctx context.Context, addr string) (net.Conn, error) {
	dialer := net.Dialer{}
	return dialer.DialContext(ctx, "unix", addr)
}

// Connect is a helper function to initiate a grpc client connection to server running at endpoint using tlsConfig
func Connect(endpoint string, tlsConfig *tls.Config, dialOptions ...grpc.DialOption) (*grpc.ClientConn, error) {
	proto, address, err := parseEndpoint(endpoint)
	if err != nil {
		return nil, err
	}

	dialOptions = append(dialOptions, grpc.WithConnectParams(grpc.ConnectParams{
		Backoff: backoff.Config{
			BaseDelay: time.Second,
			MaxDelay:  time.Second,
		},
	}))

	if tlsConfig != nil {
		dialOptions = append(dialOptions, grpc.WithTransportCredentials(credentials.NewTLS(tlsConfig)))
	} else {
		dialOptions = append(dialOptions, grpc.WithTransportCredentials(insecure.NewCredentials()))
	}

	dialOptions = append(dialOptions, grpc.WithNoProxy())

	if proto == "unix" {
		dialOptions = append(dialOptions, grpc.WithContextDialer(unixDialer))
	}
	// This is necessary when connecting via TCP and does not hurt
	// when using Unix domain sockets. It ensures that gRPC detects a dead connection
	// in a timely manner.
	// Code lifted from https://github.com/kubernetes-csi/csi-test/commit/6b8830bf5959a1c51c6e98fe514b22818b51eeeb
	dialOptions = append(dialOptions, grpc.WithKeepaliveParams(keepalive.ClientParameters{
		Time:                30 * time.Second,
		PermitWithoutStream: true,
	}))

	// Disable gRPC service config discovery to prevent DNS flooding in Kubernetes
	dialOptions = append(dialOptions, grpc.WithDisableServiceConfig())

	return grpc.NewClient(address, dialOptions...)
}

// NewServer is a helper function to start a grpc server at the given endpoint.
func NewServer(endpoint string, tlsConfig *tls.Config, opts ...grpc.ServerOption) (*grpc.Server, net.Listener, error) {
	proto, addr, err := parseEndpoint(endpoint)
	if err != nil {
		return nil, nil, err
	}

	if proto == "unix" {
		if err = os.Remove(addr); err != nil && !os.IsNotExist(err) {
			return nil, nil, err
		}
	}

	listener, err := net.Listen(proto, addr)
	if err != nil {
		return nil, nil, err
	}

	if tlsConfig != nil {
		opts = append(opts, grpc.Creds(credentials.NewTLS(tlsConfig)))
	}

	return grpc.NewServer(opts...), listener, nil
}

// ServerTLS prepares the TLS configuration needed for a server with given
// encoded certificate and private key.
func ServerTLS(caCert, cert, key []byte, peerName string) (*tls.Config, error) {
	certPool := x509.NewCertPool()
	if ok := certPool.AppendCertsFromPEM(caCert); !ok {
		return nil, fmt.Errorf("failed to append CA certificate to pool")
	}

	certificate, err := tls.X509KeyPair(cert, key)
	if err != nil {
		return nil, err
	}

	return serverConfig(certPool, &certificate, peerName), nil
}

// LoadServerTLS prepares the TLS configuration needed for a server with the given certificate files.
// peerName is either the name that the client is expected to have a certificate for or empty,
// in which case any client is allowed to connect.
func LoadServerTLS(caFile, certFile, keyFile, peerName string) (*tls.Config, error) {
	certPool, peerCert, err := loadCertificate(caFile, certFile, keyFile)
	if err != nil {
		return nil, err
	}
	return serverConfig(certPool, peerCert, peerName), nil
}

func serverConfig(certPool *x509.CertPool, peerCert *tls.Certificate, peerName string) *tls.Config {
	return &tls.Config{
		GetConfigForClient: func(info *tls.ClientHelloInfo) (*tls.Config, error) {
			if info == nil {
				return nil, errors.New("nil client info passed")
			}

			config := &tls.Config{
				MinVersion:    tls.VersionTLS13,
				Renegotiation: tls.RenegotiateNever,
				Certificates:  []tls.Certificate{*peerCert},
				ClientCAs:     certPool,
				VerifyPeerCertificate: func(rawCerts [][]byte, verifiedChains [][]*x509.Certificate) error {
					// Common name check when accepting a connection from a client.
					if peerName == "" {
						// All names allowed.
						return nil
					}

					if len(verifiedChains) == 0 ||
						len(verifiedChains[0]) == 0 {
						return errors.New("no valid certificate")
					}

					for _, name := range verifiedChains[0][0].DNSNames {
						if name == peerName {
							return nil
						}
					}
					// For compatibility - using CN as hostName
					commonName := verifiedChains[0][0].Subject.CommonName
					if commonName == peerName {
						return nil
					}
					return fmt.Errorf("certificate is not signed for %q hostname", peerName)
				},
			}
			if peerName != "" {
				config.ClientAuth = tls.RequireAndVerifyClientCert
			}
			return config, nil
		},
	}
}

// ClientTLS prepares the TLS configuration that can be used by a client while connecting to a server
// with given encoded certificate and private key.
// peerName must be provided when expecting the server to offer a certificate with that CommonName.
func ClientTLS(caCert, cert, key []byte, peerName string) (*tls.Config, error) {
	certPool := x509.NewCertPool()
	if ok := certPool.AppendCertsFromPEM(caCert); !ok {
		return nil, fmt.Errorf("failed to append CA certificate to pool")
	}

	certificate, err := tls.X509KeyPair(cert, key)
	if err != nil {
		return nil, err
	}

	return clientConfig(certPool, &certificate, peerName), nil
}

// LoadClientTLS prepares the TLS configuration that can be used by a client while connecting to a server.
// peerName must be provided when expecting the server to offer a certificate with that CommonName. caFile, certFile, and keyFile are all optional.
func LoadClientTLS(caFile, certFile, keyFile, peerName string) (*tls.Config, error) {
	certPool, peerCert, err := loadCertificate(caFile, certFile, keyFile)
	if err != nil {
		return nil, err
	}

	return clientConfig(certPool, peerCert, peerName), nil
}

func clientConfig(certPool *x509.CertPool, peerCert *tls.Certificate, peerName string) *tls.Config {
	tlsConfig := &tls.Config{
		MinVersion:    tls.VersionTLS13,
		Renegotiation: tls.RenegotiateNever,
		ServerName:    peerName,
		RootCAs:       certPool,
	}
	if peerCert != nil {
		tlsConfig.Certificates = append(tlsConfig.Certificates, *peerCert)
	}
	return tlsConfig
}

func loadCertificate(caFile, certFile, keyFile string) (certPool *x509.CertPool, peerCert *tls.Certificate, err error) {
	if certFile != "" || keyFile != "" {
		cert, err := tls.LoadX509KeyPair(certFile, keyFile)
		if err != nil {
			return nil, nil, err
		}
		peerCert = &cert
	}

	if caFile != "" {
		caCert, err := os.ReadFile(caFile)
		if err != nil {
			return nil, nil, err
		}

		certPool = x509.NewCertPool()
		if ok := certPool.AppendCertsFromPEM(caCert); !ok {
			return nil, nil, fmt.Errorf("failed to append certs from %s", caFile)
		}
	}

	return
}

// parseEndpoint splits ep string into proto and address
// the supported protocols are "tcp" and "unix" unsupported protocols will return error
// if the ep (url) does not contain a protocol, we return "tcp" for backwards compatibility
func parseEndpoint(ep string) (proto string, address string, err error) {
	s := strings.SplitN(ep, "://", 2)
	if len(s) == 0 {
		return "", "", fmt.Errorf("invalid endpoint: %v", ep)
	}

	if len(s) > 1 && s[1] != "" {
		if s[0] != "unix" && s[0] != "tcp" {
			return "", "", fmt.Errorf("invalid endpoint: %v unsupported proto: %v", ep, s[0])
		}
		return s[0], s[1], nil
	}

	// default protocol tcp to be backwards compatible
	return "tcp", s[0], nil
}
</file>

<file path="pkg/util/log.go">
package util

import (
	"bufio"
	"bytes"
	"errors"
	"fmt"
	"os"
	"path"
	"path/filepath"
	"runtime"
	"time"

	"github.com/sirupsen/logrus"
)

const (
	LogComponentField = "component"
)

type LonghornFormatter struct {
	*logrus.TextFormatter

	LogsDir string
}

type LonghornWriter struct {
	file *os.File
	name string
	path string
}

func NewLonghornWriter(name string, logsDir string) (*LonghornWriter, error) {
	logPath := filepath.Join(logsDir, name+".log")
	logPath, err := filepath.Abs(logPath)
	if err != nil {
		return nil, err
	}
	file, err := os.OpenFile(logPath, os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
	if err != nil {
		return nil, err
	}
	return &LonghornWriter{
		file: file,
		name: name,
		path: logPath,
	}, nil
}

func SetUpLogger(logsDir string) error {
	if err := os.MkdirAll(logsDir, 0755); err != nil {
		return err
	}
	logsDir, err := filepath.Abs(logsDir)
	if err != nil {
		return err
	}
	testFile := filepath.Join(logsDir, "test")
	if _, err := os.OpenFile(testFile, os.O_WRONLY|os.O_CREATE, 0644); os.IsPermission(err) {
		return err
	}
	logrus.Infof("Storing process logs at path: %v", logsDir)
	logrus.SetReportCaller(true)

	logrus.SetFormatter(LonghornFormatter{
		TextFormatter: &logrus.TextFormatter{
			DisableColors: false,
			CallerPrettyfier: func(f *runtime.Frame) (function string, file string) {
				fileName := fmt.Sprintf("%s:%d", path.Base(f.File), f.Line)
				funcName := path.Base(f.Function)
				return funcName, fileName
			},
			TimestampFormat: time.RFC3339Nano,
			FullTimestamp:   true,
		},
		LogsDir: logsDir,
	})
	return nil
}

func (l LonghornFormatter) Format(entry *logrus.Entry) ([]byte, error) {
	logMsg := &bytes.Buffer{}
	component, ok := entry.Data[LogComponentField]
	if !ok {
		component = "longhorn-instance-manager"
	}
	component, ok = component.(string)
	if !ok {
		return nil, errors.New("field component must be a string")
	}
	logMsg.WriteString("[" + component.(string) + "] ")
	if component == "longhorn-instance-manager" {
		msg, err := l.TextFormatter.Format(entry)
		if err != nil {
			return nil, err
		}
		logMsg.Write(msg)
	} else {
		logMsg.WriteString(entry.Message)
	}

	return logMsg.Bytes(), nil
}

func (l LonghornWriter) Close() error {
	if err := l.file.Close(); err != nil {
		return err
	}
	return nil
}

func (l LonghornWriter) StreamLog(done chan struct{}) (chan string, error) {
	file, err := os.OpenFile(l.path, os.O_RDONLY, 0644)
	if err != nil {
		return nil, err
	}
	logChan := make(chan string)
	scanner := bufio.NewScanner(file)
	go func() {
		for scanner.Scan() {
			select {
			case <-done:
				close(logChan)
				return
			default:
				logChan <- scanner.Text()
			}
		}
		close(logChan)
		if closeErr := file.Close(); closeErr != nil {
			logrus.WithError(closeErr).Warn("Failed to close file")
		}
	}()
	return logChan, nil
}

func (l LonghornWriter) Write(input []byte) (int, error) {
	msg := string(input)
	logrus.WithField(LogComponentField, l.name).Println(msg)
	outLen, err := l.file.Write(input)
	if err != nil {
		return 0, err
	}
	if err := l.file.Sync(); err != nil {
		return 0, err
	}
	return outLen, nil
}
</file>

<file path="pkg/util/util.go">
package util

import (
	"bytes"
	"encoding/json"
	"fmt"
	"net"
	"os"
	"os/exec"
	"regexp"
	"strconv"
	"strings"
	"time"

	"github.com/cockroachdb/errors"
	"github.com/google/uuid"
	"github.com/sirupsen/logrus"

	"k8s.io/mount-utils"

	spdkhelpertypes "github.com/longhorn/go-spdk-helper/pkg/types"
)

const (
	DefaulCmdTimeout = time.Minute // one minute by default

	GRPCHealthProbe = "/usr/local/bin/grpc_health_probe"
)

func Execute(binary string, args ...string) (string, error) {
	return ExecuteWithTimeout(DefaulCmdTimeout, binary, args...)
}

func ExecuteWithTimeout(timeout time.Duration, binary string, args ...string) (string, error) {
	var err error
	cmd := exec.Command(binary, args...)
	done := make(chan struct{})

	var output, stderr bytes.Buffer
	cmd.Stdout = &output
	cmd.Stderr = &stderr

	go func() {
		err = cmd.Run()
		done <- struct{}{}
	}()

	select {
	case <-done:
	case <-time.After(timeout):
		if cmd.Process != nil {
			if err := cmd.Process.Kill(); err != nil {
				logrus.WithError(err).Warnf("Problem killing process pid=%v", cmd.Process.Pid)
			}

		}
		return "", errors.Wrapf(err, "timeout executing: %v %v, output %s, stderr %s",
			binary, args, output.String(), stderr.String())
	}

	if err != nil {
		return "", errors.Wrapf(err, "failed to execute: %v %v, output %s, stderr %s",
			binary, args, output.String(), stderr.String())
	}
	return output.String(), nil
}

func PrintJSON(obj interface{}) error {
	output, err := json.MarshalIndent(obj, "", "\t")
	if err != nil {
		return err
	}

	fmt.Println(string(output))
	return nil
}

func GetURL(host string, port int) string {
	return net.JoinHostPort(host, strconv.Itoa(port))
}

func RemoveFile(file string) error {
	if _, err := os.Stat(file); os.IsNotExist(err) {
		// file doesn't exist
		return nil
	}

	if _, err := Execute("rm", file); err != nil {
		return errors.Wrapf(err, "failed to remove file %v", file)
	}

	return nil
}

func GRPCServiceReadinessProbe(address string) bool {
	if _, err := Execute(GRPCHealthProbe, "-addr", address); err != nil {
		return false
	}
	return true
}

func Now() string {
	return time.Now().UTC().Format(time.RFC3339)
}

func UUID() string {
	return uuid.New().String()
}

func ParsePortRange(portRange string) (int32, int32, error) {
	parts := strings.Split(portRange, "-")
	if len(parts) != 2 {
		return 0, 0, fmt.Errorf("invalid format for SPDK port range %s", portRange)
	}

	portStart, err := strconv.Atoi(strings.TrimSpace(parts[0]))
	if err != nil {
		return 0, 0, err
	}

	portEnd, err := strconv.Atoi(strings.TrimSpace(parts[1]))
	if err != nil {
		return 0, 0, err
	}

	return int32(portStart), int32(portEnd), nil
}

// IsSPDKTgtReady checks if SPDK target is ready
func IsSPDKTgtReady(timeout time.Duration) bool {
	for i := 0; i < int(timeout.Seconds()); i++ {
		conn, err := net.DialTimeout(spdkhelpertypes.DefaultJSONServerNetwork, spdkhelpertypes.DefaultUnixDomainSocketPath, 1*time.Second)
		if err == nil {
			if closeErr := conn.Close(); closeErr != nil {
				logrus.WithError(closeErr).Warn("Failed to close connection")
			}
			return true
		}
		time.Sleep(time.Second)
	}
	return false
}

func IsMountPointReadOnly(mp mount.MountPoint) bool {
	for _, opt := range mp.Opts {
		if opt == "ro" {
			return true
		}
	}
	return false
}

func GetVolumeMountPointMap() (map[string]mount.MountPoint, error) {
	volumeMountPointMap := make(map[string]mount.MountPoint)

	mounter := mount.New("")
	mountPoints, err := mounter.List()
	if err != nil {
		return nil, err
	}

	regex := regexp.MustCompile(`.*/globalmount$`)

	for _, mp := range mountPoints {
		if regex.MatchString(mp.Path) {
			volumeNameSHAStr := GetVolumeNameSHAStrFromPath(mp.Path)
			volumeMountPointMap[volumeNameSHAStr] = mp
		}
	}
	return volumeMountPointMap, nil
}

func GetVolumeNameSHAStrFromPath(path string) string {
	// mount path for volume: "/host/var/lib/kubelet/plugins/kubernetes.io/csi/driver.longhorn.io/${VolumeNameSHAStr}/globalmount"
	pathSlices := strings.Split(path, "/")
	volumeNameSHAStr := pathSlices[len(pathSlices)-2]
	return volumeNameSHAStr
}

func ProcessNameToVolumeName(processName string) string {
	// process name: "pvc-e130e369-274d-472d-98d1-f6074d2725e8-e-0"
	nameSlices := strings.Split(processName, "-")
	volumeName := strings.Join(nameSlices[:len(nameSlices)-2], "-")
	return volumeName
}
</file>

<file path="scripts/build">
#!/bin/bash
set -e

source $(dirname $0)/version

LINKFLAGS="-X main.Version=$VERSION
           -X main.GitCommit=$GITCOMMIT
           -X main.BuildDate=$BUILDDATE
           -linkmode external -extldflags -static"

# add coverage flags if there is no tag and it's on master or a version branch like v1.6.x
COMMIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
COMMIT_TAG=$(git tag --points-at HEAD | head -n 1)
if [[ "$COMMIT_TAG" == "" ]] && [[ "$COMMIT_BRANCH" == master || "$COMMIT_BRANCH" =~ ^v[0-9]+\.[0-9]+\.x$ ]]; then
    COVER="-cover"
    COVERPKG="-coverpkg=github.com/longhorn/longhorn-instance-manager/..."
fi

cd $(dirname $0)/..

mkdir -p bin
go build -o bin/longhorn-instance-manager -tags netgo -ldflags "$LINKFLAGS" $COVER $COVERPKG
</file>

<file path="scripts/ci">
#!/bin/bash
set -e

cd $(dirname $0)

SKIP_TASKS=${SKIP_TASKS:-}
IFS=' ' read -ra skip_tasks <<<"$SKIP_TASKS"

tasks=("build" "validate" "test" "package")

for task in "${tasks[@]}"; do
  for skip_task in "${skip_tasks[@]}"; do
    if [ "$task" = "$skip_task" ]; then
      continue 2
    fi
  done

  ./"$task"
done
</file>

<file path="scripts/entry">
#!/bin/bash
set -e

trap "chown -R $DAPPER_UID:$DAPPER_GID ." exit

export GOFLAGS=-mod=vendor

mkdir -p bin
if [ -e ./scripts/$1 ]; then
    ./scripts/"$@"
else
    "$@"
fi
</file>

<file path="scripts/package">
#!/bin/bash
set -e

source $(dirname $0)/version

cd $(dirname $0)/..

PROJECT=$(basename "${PWD}")

command -v buildx >/dev/null && BUILD_CMD=(buildx) || BUILD_CMD=(docker buildx)

# read configurable parameters
REPO=${REPO:-longhornio}
IMAGE_NAME=${IMAGE_NAME:-${PROJECT}}
TAG=${TAG:-''}
PUSH=${PUSH:-'false'}
IS_SECURE=${IS_SECURE:-'false'}
MACHINE=${MACHINE:-''}
TARGET_PLATFORMS=${TARGET_PLATFORMS:-''}
IID_FILE=${IID_FILE:-''}
IID_FILE_FLAG=${IID_FILE_FLAG:-''}
SRC_BRANCH=${SRC_BRANCH:-''}
SRC_TAG=${SRC_TAG:-''}

# TODO: implement self-contained build
[[ ! -x ./bin/longhorn-instance-manager ]] && ./scripts/build

if [[ -z $TAG ]]; then
    if API_VERSION=$(./bin/longhorn-instance-manager version --client-only | jq ".clientVersion.instanceManagerAPIVersion"); then
      TAG="v${API_VERSION}_$(date -u +%Y%m%d)"
    else
      TAG="${VERSION}"
    fi
fi

IMAGE="${REPO}/${IMAGE_NAME}:${TAG}"

BUILDER_ARGS=()
[[ ${MACHINE} ]] && BUILDER_ARGS+=('--builder' "${MACHINE}")

IFS=' ' read -r -a IID_FILE_ARGS <<<"$IID_FILE_FLAG"
[[ -n "${IID_FILE}" && ${#IID_FILE_ARGS} == 0 ]] && IID_FILE_ARGS=('--iidfile' "${IID_FILE}")

BUILDX_ARGS=()

if [[ "${PUSH}" == 'true' ]]; then
    BUILDX_ARGS+=('--push')
else
    BUILDX_ARGS+=('--load')
fi

[[ ${IS_SECURE} == 'true' ]] && BUILDX_ARGS+=('--sbom=true' '--attest' 'type=provenance,mode=max')

if [[ ${TARGET_PLATFORMS} ]] ; then
    IFS='/' read -r OS ARCH <<<"${TARGET_PLATFORMS}"
    BUILDX_ARGS+=('--platform' "${TARGET_PLATFORMS}")
else
    case $(uname -m) in
    aarch64 | arm64)
        ARCH=arm64
        ;;
    x86_64)
        ARCH=amd64
        ;;
    *)
        echo "$(uname -a): unsupported architecture"
        exit 1
    esac
    BUILDX_ARGS+=('--platform' "linux/${ARCH}")
fi

IMAGE_ARGS=(--build-arg ARCH="${ARCH}")
[[ -n "${SRC_BRANCH}" ]] && IMAGE_ARGS+=(--build-arg SRC_BRANCH="${SRC_BRANCH}")
[[ -n "${SRC_TAG}" ]] && IMAGE_ARGS+=(--build-arg SRC_TAG="${SRC_TAG}")

# update base image to get latest changes
grep 'FROM.*/' package/Dockerfile | awk '{print $2}' | while read -r BASE_IMAGE
do
    docker pull "${BASE_IMAGE}"
done

echo "Building ${IMAGE} with ARCH=${ARCH} SRC_BRANCH=${SRC_BRANCH} SRC_TAG=${SRC_TAG}"
IMAGE_BUILD_CMD_ARGS=(
    build --no-cache \
    "${BUILDER_ARGS[@]}" \
    "${IID_FILE_ARGS[@]}" \
    "${BUILDX_ARGS[@]}" \
    "${IMAGE_ARGS[@]}" \
    -t "${IMAGE}" -f package/Dockerfile .
)
echo "${BUILD_CMD[@]}" "${IMAGE_BUILD_CMD_ARGS[@]}"
"${BUILD_CMD[@]}" "${IMAGE_BUILD_CMD_ARGS[@]}"

echo "Built ${IMAGE}"

mkdir -p ./bin
echo "${IMAGE}" > ./bin/latest_image
</file>

<file path="scripts/test">
#!/bin/bash
set -e

cd $(dirname $0)/..

echo Running tests

PACKAGES="$(find . -name '*.go' | xargs -I{} dirname {} | sort -u | grep -Ev '(.git|.trash-cache|vendor|bin)')"

echo Packages: ${PACKAGES}

[ "${ARCH}" == "amd64" ] && RACE=-race
go test ${RACE} -coverprofile=coverage.out ${PACKAGES}
</file>

<file path="scripts/validate">
#!/bin/bash
set -e

cd $(dirname $0)/..

echo Running validation

PACKAGES="$(find -name '*.go' | xargs -I{} dirname {} |  cut -f2 -d/ | sort -u | grep -Ev '(^\.$|.git|.trash-cache|vendor|bin)' | sed -e 's!^!./!' -e 's!$!/...!')"

echo Running: go vet
go vet ${PACKAGES}

echo "Running: golangci-lint"
golangci-lint run --timeout=5m

echo Running: go fmt
test -z "$(go fmt ${PACKAGES} | tee /dev/stderr)"
</file>

<file path="scripts/version">
#!/bin/bash

if [ -n "$(git status --porcelain --untracked-files=no)" ]; then
    DIRTY="-dirty"
fi

COMMIT=$(git rev-parse --short HEAD)
GIT_TAG=$(git tag -l --contains HEAD | head -n 1)

if [[ -z "$DIRTY" && -n "$GIT_TAG" ]]; then
    VERSION=$GIT_TAG
else
    VERSION="${COMMIT}${DIRTY}"
fi

GITCOMMIT=$(git rev-parse HEAD)
BUILDDATE=$(date -u --rfc-3339=seconds)
BUILDDATE=${BUILDDATE// /T}
</file>

<file path=".gitignore">
# Binaries for programs and plugins
*.exe
*.dll
*.so
*.dylib
.dapper
Dockerfile.dapper[0-9]*

# Editors
.idea
.vscode

# Test binary, build with `go test -c`
*.test

# Output of the go coverage tool, specifically when used with LiteIDE
*.out

# vim temp file
.*.swp

# Misc
bin/
</file>

<file path="CODE_OF_CONDUCT.md">
# Longhorn Community Code of Conduct

Longhorn follows the [Cloud Native Computing Foundation Code of Conduct](https://github.com/cncf/foundation/blob/master/code-of-conduct.md).
</file>

<file path="codecov.yml">
comment: off
coverage:
  status:
    project:
      default:
        informational: true
    patch: off
</file>

<file path="Dockerfile.dapper">
FROM registry.suse.com/bci/golang:1.25

ARG DAPPER_HOST_ARCH
ARG http_proxy
ARG https_proxy
ARG SRC_BRANCH=master
ARG SRC_TAG
ARG CACHEBUST

ENV HOST_ARCH=${DAPPER_HOST_ARCH} ARCH=${DAPPER_HOST_ARCH}
ENV DAPPER_DOCKER_SOCKET true
ENV DAPPER_ENV TAG REPO DRONE_REPO DRONE_PULL_REQUEST DRONE_COMMIT_REF SKIP_TASKS
ENV DAPPER_OUTPUT bin coverage.out
ENV DAPPER_RUN_ARGS --privileged --tmpfs /go/src/github.com/longhorn/longhorn-engine/integration/.venv:exec --tmpfs /go/src/github.com/longhorn/longhorn-engine/integration/.tox:exec -v /dev:/host/dev -v /proc:/host/proc
ENV DAPPER_SOURCE /go/src/github.com/longhorn/longhorn-instance-manager
ENV SRC_BRANCH ${SRC_BRANCH}
ENV SRC_TAG ${SRC_TAG}

WORKDIR ${DAPPER_SOURCE}

ENTRYPOINT ["./scripts/entry"]
CMD ["ci"]


RUN zypper -n ref && \
    zypper update -y

# Install packages
RUN zypper -n install cmake wget curl git less file \
    libglib-2_0-0 libkmod-devel libnl3-devel linux-glibc-devel pkg-config \
    psmisc tox qemu-tools fuse python3-devel zlib-devel zlib-devel-static \
    bash-completion rdma-core-devel libibverbs xsltproc docbook-xsl-stylesheets \
    perl-Config-General libaio-devel glibc-devel-static glibc-devel iptables libltdl7 \
    libdevmapper1_03 iproute2 jq docker gcc gcc-c++ automake gettext gettext-tools libtool && \
    rm -rf /var/cache/zypp/*

# Install golanci-lint
RUN curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin latest

# If TAG is explicitly set and exists in the repo, switch to the tag
RUN git clone https://github.com/longhorn/dep-versions.git -b ${SRC_BRANCH} /usr/src/dep-versions && \
    cd /usr/src/dep-versions && \
    if [ -n "${SRC_TAG}" ] && git show-ref --tags ${SRC_TAG} > /dev/null 2>&1; then \
        echo "Checking out tag ${SRC_TAG}"; \
        cd /usr/src/dep-versions && git checkout tags/${SRC_TAG}; \
    fi

# Install libqcow to resolve error:
#   vendor/github.com/longhorn/longhorn-engine/pkg/qcow/libqcow.go:6:11: fatal error: libqcow.h: No such file or directory
RUN export REPO_OVERRIDE="" && \
    export COMMIT_ID_OVERRIDE="" && \
    bash /usr/src/dep-versions/scripts/build-libqcow.sh "${REPO_OVERRIDE}" "${COMMIT_ID_OVERRIDE}" && \
    ldconfig

# Docker Builx: The docker version in dapper is too old to have buildx. Install it manually.
RUN curl -sSfLO https://github.com/docker/buildx/releases/download/v0.13.1/buildx-v0.13.1.linux-${ARCH} && \
    chmod +x buildx-v0.13.1.linux-${ARCH} && \
    mv buildx-v0.13.1.linux-${ARCH} /usr/local/bin/buildx
</file>

<file path="go.mod">
module github.com/longhorn/longhorn-instance-manager

go 1.25.3

toolchain go1.26.0

require (
	github.com/cockroachdb/errors v1.12.0
	github.com/google/uuid v1.6.0
	github.com/longhorn/backupstore v0.0.0-20260127053626-a9fd84935804
	github.com/longhorn/go-common-libs v0.0.0-20260120075236-9a9dafa0a7ea
	github.com/longhorn/go-spdk-helper v0.4.1-0.20260206121825-8c1c33c29f35
	github.com/longhorn/longhorn-engine v1.12.0-dev-20260208
	github.com/longhorn/longhorn-spdk-engine v0.0.0-20260209040142-cfe6a002cffe
	github.com/longhorn/types v0.0.0-20260118234234-63598269a247
	github.com/sirupsen/logrus v1.9.4
	github.com/urfave/cli v1.22.17
	golang.org/x/sync v0.19.0
	google.golang.org/grpc v1.79.1
	google.golang.org/protobuf v1.36.11
	gopkg.in/check.v1 v1.0.0-20201130134442-10cb98267c6c
	k8s.io/mount-utils v0.35.1
)

require (
	github.com/0xPolygon/polygon-edge v1.3.3 // indirect
	github.com/Azure/azure-sdk-for-go/sdk/azcore v1.16.0 // indirect
	github.com/Azure/azure-sdk-for-go/sdk/internal v1.10.0 // indirect
	github.com/Azure/azure-sdk-for-go/sdk/storage/azblob v1.5.0 // indirect
	github.com/RoaringBitmap/roaring v1.9.4 // indirect
	github.com/aws/aws-sdk-go-v2 v1.39.6 // indirect
	github.com/aws/aws-sdk-go-v2/aws/protocol/eventstream v1.7.1 // indirect
	github.com/aws/aws-sdk-go-v2/config v1.31.20 // indirect
	github.com/aws/aws-sdk-go-v2/credentials v1.18.24 // indirect
	github.com/aws/aws-sdk-go-v2/feature/ec2/imds v1.18.13 // indirect
	github.com/aws/aws-sdk-go-v2/feature/s3/manager v1.19.8 // indirect
	github.com/aws/aws-sdk-go-v2/feature/s3/transfermanager v0.1.0 // indirect
	github.com/aws/aws-sdk-go-v2/internal/configsources v1.4.13 // indirect
	github.com/aws/aws-sdk-go-v2/internal/endpoints/v2 v2.7.13 // indirect
	github.com/aws/aws-sdk-go-v2/internal/ini v1.8.4 // indirect
	github.com/aws/aws-sdk-go-v2/internal/v4a v1.4.8 // indirect
	github.com/aws/aws-sdk-go-v2/service/internal/accept-encoding v1.13.3 // indirect
	github.com/aws/aws-sdk-go-v2/service/internal/checksum v1.8.8 // indirect
	github.com/aws/aws-sdk-go-v2/service/internal/presigned-url v1.13.13 // indirect
	github.com/aws/aws-sdk-go-v2/service/internal/s3shared v1.19.8 // indirect
	github.com/aws/aws-sdk-go-v2/service/s3 v1.88.2 // indirect
	github.com/aws/aws-sdk-go-v2/service/sso v1.30.3 // indirect
	github.com/aws/aws-sdk-go-v2/service/ssooidc v1.35.7 // indirect
	github.com/aws/aws-sdk-go-v2/service/sts v1.40.2 // indirect
	github.com/aws/smithy-go v1.23.2 // indirect
	github.com/beorn7/perks v1.0.1 // indirect
	github.com/bits-and-blooms/bitset v1.16.0 // indirect
	github.com/c9s/goprocinfo v0.0.0-20210130143923-c95fcf8c64a8 // indirect
	github.com/cespare/xxhash/v2 v2.3.0 // indirect
	github.com/cockroachdb/logtags v0.0.0-20230118201751-21c54148d20b // indirect
	github.com/cockroachdb/redact v1.1.5 // indirect
	github.com/cpuguy83/go-md2man/v2 v2.0.7 // indirect
	github.com/davecgh/go-spew v1.1.2-0.20180830191138-d8f796af33cc // indirect
	github.com/emicklei/go-restful/v3 v3.12.2 // indirect
	github.com/felixge/httpsnoop v1.0.4 // indirect
	github.com/fxamacker/cbor/v2 v2.9.0 // indirect
	github.com/gammazero/deque v1.0.0 // indirect
	github.com/gammazero/workerpool v1.1.3 // indirect
	github.com/getsentry/sentry-go v0.27.0 // indirect
	github.com/go-logr/logr v1.4.3 // indirect
	github.com/go-ole/go-ole v1.3.0 // indirect
	github.com/go-openapi/jsonpointer v0.21.0 // indirect
	github.com/go-openapi/jsonreference v0.20.2 // indirect
	github.com/go-openapi/swag v0.23.0 // indirect
	github.com/gofrs/flock v0.13.0 // indirect
	github.com/gogo/protobuf v1.3.2 // indirect
	github.com/google/gnostic-models v0.7.0 // indirect
	github.com/gorilla/handlers v1.5.2 // indirect
	github.com/jinzhu/copier v0.4.0 // indirect
	github.com/josharian/intern v1.0.0 // indirect
	github.com/json-iterator/go v1.1.12 // indirect
	github.com/kr/pretty v0.3.1 // indirect
	github.com/kr/text v0.2.0 // indirect
	github.com/longhorn/go-iscsi-helper v0.0.0-20260125095104-688e170e56e5 // indirect
	github.com/longhorn/sparse-tools v0.0.0-20260117144214-070853c24eda // indirect
	github.com/mailru/easyjson v0.7.7 // indirect
	github.com/mitchellh/go-ps v1.0.0 // indirect
	github.com/moby/sys/mountinfo v0.7.2 // indirect
	github.com/modern-go/concurrent v0.0.0-20180306012644-bacd9c7ef1dd // indirect
	github.com/modern-go/reflect2 v1.0.3-0.20250322232337-35a7c28c31ee // indirect
	github.com/mschoch/smat v0.2.0 // indirect
	github.com/munnerz/goautoneg v0.0.0-20191010083416-a7dc8b61c822 // indirect
	github.com/pierrec/lz4/v4 v4.1.25 // indirect
	github.com/pkg/errors v0.9.1 // indirect
	github.com/power-devops/perfstat v0.0.0-20240221224432-82ca36839d55 // indirect
	github.com/prometheus/client_golang v1.20.5 // indirect
	github.com/prometheus/client_model v0.6.1 // indirect
	github.com/prometheus/common v0.60.1 // indirect
	github.com/prometheus/procfs v0.15.1 // indirect
	github.com/rancher/go-fibmap v0.0.0-20160418233256-5fc9f8c1ed47 // indirect
	github.com/rogpeppe/go-internal v1.14.1 // indirect
	github.com/russross/blackfriday/v2 v2.1.0 // indirect
	github.com/shirou/gopsutil/v3 v3.24.5 // indirect
	github.com/slok/goresilience v0.2.0 // indirect
	github.com/x448/float16 v0.8.4 // indirect
	github.com/yusufpapurcu/wmi v1.2.4 // indirect
	go.uber.org/multierr v1.11.0 // indirect
	go.yaml.in/yaml/v2 v2.4.3 // indirect
	go.yaml.in/yaml/v3 v3.0.4 // indirect
	golang.org/x/exp v0.0.0-20260112195511-716be5621a96 // indirect
	golang.org/x/net v0.49.0 // indirect
	golang.org/x/oauth2 v0.34.0 // indirect
	golang.org/x/sys v0.40.0 // indirect
	golang.org/x/term v0.39.0 // indirect
	golang.org/x/text v0.33.0 // indirect
	golang.org/x/time v0.9.0 // indirect
	google.golang.org/genproto/googleapis/rpc v0.0.0-20251202230838-ff82c1b0f217 // indirect
	gopkg.in/evanphx/json-patch.v4 v4.13.0 // indirect
	gopkg.in/inf.v0 v0.9.1 // indirect
	gopkg.in/yaml.v3 v3.0.1 // indirect
	k8s.io/api v0.35.0 // indirect
	k8s.io/apimachinery v0.35.0 // indirect
	k8s.io/client-go v0.35.0 // indirect
	k8s.io/klog/v2 v2.130.1 // indirect
	k8s.io/kube-openapi v0.0.0-20250910181357-589584f1c912 // indirect
	k8s.io/utils v0.0.0-20251002143259-bc988d571ff4 // indirect
	sigs.k8s.io/json v0.0.0-20250730193827-2d320260d730 // indirect
	sigs.k8s.io/randfill v1.0.0 // indirect
	sigs.k8s.io/structured-merge-diff/v6 v6.3.0 // indirect
	sigs.k8s.io/yaml v1.6.0 // indirect
)
</file>

<file path="LICENSE">
Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding those notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. We also recommend that a
      file or class name and description of purpose be included on the
      same "printed page" as the copyright notice for easier
      identification within third-party archives.

   Copyright [yyyy] [name of copyright owner]

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
</file>

<file path="main.go">
package main

import (
	"fmt"
	"os"
	"path"
	"runtime"
	"time"

	"github.com/sirupsen/logrus"
	"github.com/urfave/cli"

	"github.com/longhorn/longhorn-instance-manager/app/cmd"
	"github.com/longhorn/longhorn-instance-manager/pkg/meta"
)

// following variables will be filled by `-ldflags "-X ..."`
var (
	Version   string
	GitCommit string
	BuildDate string
)

func main() {
	a := cli.NewApp()

	a.Version = Version
	meta.Version = Version
	meta.GitCommit = GitCommit
	meta.BuildDate = BuildDate

	logrus.SetReportCaller(true)
	logrus.SetFormatter(&logrus.TextFormatter{
		CallerPrettyfier: func(f *runtime.Frame) (function string, file string) {
			fileName := fmt.Sprintf("%s:%d", path.Base(f.File), f.Line)
			funcName := path.Base(f.Function)
			return funcName, fileName
		},
		TimestampFormat: time.RFC3339Nano,
		FullTimestamp:   true,
	})

	a.Before = func(c *cli.Context) error {
		if c.GlobalBool("debug") {
			logrus.SetLevel(logrus.DebugLevel)
		}
		return nil
	}
	a.Flags = []cli.Flag{
		cli.StringFlag{
			Name:  "url",
			Value: "tcp://localhost:8500",
			Usage: "specifies the server endpoint to connect to supported protocols are 'tcp' and 'unix'",
		},
		cli.BoolFlag{
			Name: "debug",
		},
		cli.StringFlag{
			Name:     "tls-dir",
			Usage:    "when present will look for `tls.crt` and `tls.key` and `ca.crt` file in the specified directory",
			EnvVar:   "TLS_DIR",
			Required: false,
		},
	}
	a.Commands = []cli.Command{
		cmd.StartCmd(),
		cmd.ProcessCmd(),
		cmd.VersionCmd(),
	}
	if err := a.Run(os.Args); err != nil {
		logrus.WithError(err).Fatal("Error when executing command")
	}
}
</file>

<file path="Makefile">
PROJECT := longhorn-instance-manager
TARGETS := $(shell ls scripts)
MACHINE := longhorn
# Define the target platforms that can be used across the ecosystem.
# Note that what would actually be used for a given project will be
# defined in TARGET_PLATFORMS, and must be a subset of the below:
DEFAULT_PLATFORMS := linux/amd64,linux/arm64

export SRC_BRANCH := $(shell bash -c 'source <(curl -s "https://raw.githubusercontent.com/longhorn/dep-versions/master/scripts/common.sh") && get_branch')
export SRC_TAG := $(shell git tag --points-at HEAD | head -n 1)

export CACHEBUST := $(shell date +%s)

.dapper:
	@echo Downloading dapper
	@curl -sL https://releases.rancher.com/dapper/latest/dapper-`uname -s`-`uname -m` > .dapper.tmp
	@@chmod +x .dapper.tmp
	@./.dapper.tmp -v
	@mv .dapper.tmp .dapper

$(TARGETS): .dapper
	./.dapper $@

.PHONY: buildx-machine
buildx-machine:
	@docker buildx create --name=$(MACHINE) --platform=$(DEFAULT_PLATFORMS) 2>/dev/null || true
	docker buildx inspect $(MACHINE)

# variables needed from GHA caller:
# - REPO: image repo, include $registry/$repo_path
# - TAG: image tag
# - TARGET_PLATFORMS: optional, to be passed for buildx's --platform option
# - IID_FILE_FLAG: optional, options to generate image ID file
.PHONY: workflow-image-build-push workflow-image-build-push-secure workflow-manifest-image
workflow-image-build-push: buildx-machine
	MACHINE=$(MACHINE) PUSH='true' IMAGE_NAME=$(PROJECT) bash scripts/package
workflow-image-build-push-secure: buildx-machine
	MACHINE=$(MACHINE) PUSH='true' IMAGE_NAME=$(PROJECT) IS_SECURE=true bash scripts/package
workflow-manifest-image:
	docker pull --platform linux/amd64 ${REPO}/longhorn-instance-manager:${TAG}-amd64
	docker pull --platform linux/arm64 ${REPO}/longhorn-instance-manager:${TAG}-arm64
	docker buildx imagetools create -t ${REPO}/longhorn-instance-manager:${TAG} \
	  ${REPO}/longhorn-instance-manager:${TAG}-amd64 \
	  ${REPO}/longhorn-instance-manager:${TAG}-arm64

trash: .dapper
	./.dapper -m bind trash

trash-keep: .dapper
	./.dapper -m bind trash -k

deps: trash

.DEFAULT_GOAL := ci

.PHONY: $(TARGETS)
</file>

<file path="README.md">
# Longhorn Instance Manager

[![Build Status](https://github.com/longhorn/longhorn-instance-manager/actions/workflows/build.yml/badge.svg)](https://github.com/longhorn/longhorn-instance-manager/actions/workflows/build.yml)[![Go Report Card](https://goreportcard.com/badge/github.com/longhorn/longhorn-instance-manager)](https://goreportcard.com/report/github.com/longhorn/longhorn-instance-manager)

Longhorn Instance Manager manages engine and replica instances on the node.
</file>

<file path="renovate.json">
{
  "extends": ["github>longhorn/release:renovate-default"]
}
</file>

<file path="version">
v1.12.0-dev
</file>

</files>
