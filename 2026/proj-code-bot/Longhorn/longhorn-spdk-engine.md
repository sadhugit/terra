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
    backport-pr.yml
    build.yml
    codespell.yml
    conventional_commits.yml
    fossa.yml
    stale.yaml
  mergify.yml
  PULL_REQUEST_TEMPLATE.md
pkg/
  api/
    types.go
  client/
    client_backing_image.go
    client_disk.go
    client_engine_frontend.go
    client_engine.go
    client_log.go
    client_replica.go
    client.go
    types.go
  log/
    log.go
  spdk/
    disk/
      aio/
        aio.go
      nvme/
        nvme.go
      virtio-blk/
        virtio-blk.go
      virtio-scsi/
        virtio-scsi.go
      driver.go
      types_test.go
      types.go
    backing_image.go
    backup_restore_test.go
    backup.go
    disk.go
    engine_test.go
    engine.go
    enginefrontend_create_test.go
    enginefrontend_persist_test.go
    enginefrontend_persist.go
    enginefrontend_race_test.go
    enginefrontend.go
    expand_test.go
    log.go
    replica.go
    restore.go
    server_backingimage.go
    server_disk.go
    server_engine.go
    server_enginefrontend.go
    server_log.go
    server_replica.go
    server_verify_test.go
    server.go
    snapshot_test.go
    switchover_test.go
    types.go
    util_test.go
    util.go
  types/
    types.go
  util/
    broadcaster/
      broadcaster.go
    block_test.go
    block.go
    http_handler.go
    util.go
  spdk_test.go
scripts/
  build
  ci
  entry
  test
  validate
.gitignore
go.mod
LICENSE
Makefile
README.md
renovate.json
</directory_structure>

<files>
This section contains the contents of the repository's files.

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
    uses: longhorn/longhorn/.github/workflows/backport-pr.yml@eb3790253449e3577b4acb88b5620258cde6d747 # v1.11.1
</file>

<file path=".github/workflows/build.yml">
name: build
on:
  push:
    branches:
    - main
  pull_request:
  workflow_dispatch:
jobs:
  build-amd64:
    name: Build AMD64 binaries
    runs-on: longhorn-infra-oracle-amd64-spdk-runners
    steps:
    - name: Checkout code
      uses: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4

    - name: Install make curl git kmod
      run: |
        sudo apt update
        sudo apt-get -y install make curl git kmod
    # Build binaries
    - name: Run ci
      run: make ci

    - uses: codecov/codecov-action@b9fd7d16f6d7d1b5d2bec1a2887e65ceed900238 # v4
      with:
        files: ./coverage.out
        flags: unittests
        token: ${{ secrets.CODECOV_TOKEN }}

  build-arm64:
    name: Build ARM64 binaries
    runs-on: longhorn-infra-oracle-arm64-spdk-runners
    steps:
    - name: Checkout code
      uses: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4

    - name: Install make curl git kmod
      run: |
        sudo apt update
        sudo apt-get -y install make curl git kmod
    # Build binaries
    - name: Run ci
      run: sudo make ci

    - uses: codecov/codecov-action@b9fd7d16f6d7d1b5d2bec1a2887e65ceed900238 # v4
      with:
        files: ./coverage.out
        flags: unittests
        token: ${{ secrets.CODECOV_TOKEN }}
</file>

<file path=".github/workflows/codespell.yml">
name: Codespell

on:
  pull_request:
    branches:
    - master
    - main
    - "v*.*.*"

jobs:
  codespell:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4
      with:
        fetch-depth: 1
    - name: Check code spell
      uses: codespell-project/actions-codespell@406322ec52dd7b488e48c1c4b82e2a8b3a1bf630 # v2
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
    - uses: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4
      with:
        fetch-depth: 0
    - name: Lint Commits
      uses: wagoid/commitlint-github-action@b948419dd99f3fd78a6548d48f94e3df7f6bf3ed # v6
    - name: Lint Pull Request
      uses: amannn/action-semantic-pull-request@0723387faaf9b38adef4775cd42cfd5155ed6017 # v5.5.3
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
      - main
      - v*
    tags:
      - v*
  pull_request:
    branches:
      - master
      - main
      - v*
  workflow_dispatch: {}

permissions: {}

jobs:
  fossa-scan:
    if: github.repository == 'longhorn/longhorn-spdk-engine' # FOSSA is not intended to run on forks.
    runs-on: ubuntu-latest
    permissions:
      contents: read
    env:
      FOSSA_API_KEY: ${{ secrets.FOSSA_API_KEY }}
    steps:
      - name: "Checkout code"
        uses: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4

      - name: "Run FOSSA Scan"
        uses: fossas/fossa-action@c414b9ad82eaad041e47a7cf62a4f02411f427a0 # v1.8.0 # Use a specific version if locking is preferred
        with:
          api-key: ${{ secrets.FOSSA_API_KEY }}
          project: longhorn-spdk-engine
</file>

<file path=".github/workflows/stale.yaml">
name: 'Close stale issues and PRs'

on:
  workflow_dispatch:
  schedule:
    - cron: '30 1 * * *'

jobs:
  call-workflow:
    uses: longhorn/longhorn/.github/workflows/stale.yaml@eb3790253449e3577b4acb88b5620258cde6d747 # v1.11.1
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

<file path="pkg/api/types.go">
package api

import (
	"google.golang.org/protobuf/types/known/emptypb"

	"github.com/longhorn/types/pkg/generated/spdkrpc"

	"github.com/longhorn/longhorn-spdk-engine/pkg/types"
)

type SnapshotOptions struct {
	UserCreated bool
	Timestamp   string
}

type Replica struct {
	Name             string           `json:"name"`
	LvsName          string           `json:"lvs_name"`
	LvsUUID          string           `json:"lvs_uuid"`
	SpecSize         uint64           `json:"spec_size"`
	ActualSize       uint64           `json:"actual_size"`
	Head             *Lvol            `json:"head"`
	Snapshots        map[string]*Lvol `json:"snapshots"`
	IP               string           `json:"ip"`
	PortStart        int32            `json:"port_start"`
	PortEnd          int32            `json:"port_end"`
	State            string           `json:"state"`
	ErrorMsg         string           `json:"error_msg"`
	Rebuilding       bool             `json:"rebuilding"`
	BackingImageName string           `json:"backing_image_name"`
	UUID             string           `json:"uuid"`
}

type Lvol struct {
	Name              string          `json:"name"`
	UUID              string          `json:"uuid"`
	SpecSize          uint64          `json:"spec_size"`
	ActualSize        uint64          `json:"actual_size"`
	Parent            string          `json:"parent"`
	Children          map[string]bool `json:"children"`
	CreationTime      string          `json:"creation_time"`
	UserCreated       bool            `json:"user_created"`
	SnapshotTimestamp string          `json:"snapshot_timestamp"`
	SnapshotChecksum  string          `json:"snapshot_checksum"`
}

func ProtoLvolToLvol(l *spdkrpc.Lvol) *Lvol {
	if l == nil {
		return nil
	}
	parent := l.Parent
	if types.IsBackingImageSnapLvolName(parent) {
		parent = ""
	}
	return &Lvol{
		Name: l.Name,
		// UUID:         l.Uuid,
		SpecSize:          l.SpecSize,
		ActualSize:        l.ActualSize,
		Parent:            parent,
		Children:          l.Children,
		CreationTime:      l.CreationTime,
		UserCreated:       l.UserCreated,
		SnapshotTimestamp: l.SnapshotTimestamp,
		SnapshotChecksum:  l.SnapshotChecksum,
	}
}

func LvolToProtoLvol(l *Lvol) *spdkrpc.Lvol {
	if l == nil {
		return nil
	}
	return &spdkrpc.Lvol{
		Name: l.Name,
		// Uuid:         l.UUID,
		SpecSize:          l.SpecSize,
		ActualSize:        l.ActualSize,
		Parent:            l.Parent,
		Children:          l.Children,
		CreationTime:      l.CreationTime,
		UserCreated:       l.UserCreated,
		SnapshotTimestamp: l.SnapshotTimestamp,
		SnapshotChecksum:  l.SnapshotChecksum,
	}
}

func ProtoReplicaToReplica(r *spdkrpc.Replica) *Replica {
	res := &Replica{
		Name:       r.Name,
		LvsName:    r.LvsName,
		LvsUUID:    r.LvsUuid,
		SpecSize:   r.SpecSize,
		ActualSize: r.ActualSize,
		Head:       ProtoLvolToLvol(r.Head),
		Snapshots:  map[string]*Lvol{},
		IP:         r.Ip,
		PortStart:  r.PortStart,
		PortEnd:    r.PortEnd,
		State:      r.State,
		ErrorMsg:   r.ErrorMsg,
		Rebuilding: r.Rebuilding,
		UUID:       r.Uuid,
	}
	for snapName, snapProtoLvol := range r.Snapshots {
		res.Snapshots[snapName] = ProtoLvolToLvol(snapProtoLvol)
	}

	if r.BackingImageName != "" {
		res.BackingImageName = r.BackingImageName
	}

	return res
}

func ReplicaToProtoReplica(r *Replica) *spdkrpc.Replica {
	snapshots := map[string]*spdkrpc.Lvol{}
	for name, snapshot := range r.Snapshots {
		snapshots[name] = LvolToProtoLvol(snapshot)
	}

	res := &spdkrpc.Replica{
		Name:       r.Name,
		LvsName:    r.LvsName,
		LvsUuid:    r.LvsUUID,
		SpecSize:   r.SpecSize,
		ActualSize: r.ActualSize,
		Ip:         r.IP,
		PortStart:  r.PortStart,
		PortEnd:    r.PortEnd,
		Head:       LvolToProtoLvol(r.Head),
		Snapshots:  snapshots,
		Rebuilding: r.Rebuilding,
		State:      r.State,
		ErrorMsg:   r.ErrorMsg,
		Uuid:       r.UUID,
	}

	if r.BackingImageName != "" {
		res.BackingImageName = r.BackingImageName
	}
	return res
}

type Engine struct {
	Name                  string                `json:"name"`
	VolumeName            string                `json:"volumeName"`
	SpecSize              uint64                `json:"spec_size"`
	ActualSize            uint64                `json:"actual_size"`
	IP                    string                `json:"ip"`
	Port                  int32                 `json:"port"`
	ReplicaAddressMap     map[string]string     `json:"replica_address_map"`
	ReplicaModeMap        map[string]types.Mode `json:"replica_mode_map"`
	Head                  *Lvol                 `json:"head"`
	Snapshots             map[string]*Lvol      `json:"snapshots"`
	Frontend              string                `json:"frontend"`
	Endpoint              string                `json:"endpoint"`
	UUID                  string                `json:"uuid"`
	State                 string                `json:"state"`
	ErrorMsg              string                `json:"error_msg"`
	IsExpanding           bool                  `json:"is_expanding"`
	LastExpansionError    string                `json:"last_expansion_error"`
	LastExpansionFailedAt string                `json:"last_expansion_failed_at"`
}

func ProtoEngineToEngine(e *spdkrpc.Engine) *Engine {
	res := &Engine{
		Name:                  e.Name,
		VolumeName:            e.VolumeName,
		SpecSize:              e.SpecSize,
		ActualSize:            e.ActualSize,
		IP:                    e.Ip,
		Port:                  e.Port,
		ReplicaAddressMap:     e.ReplicaAddressMap,
		ReplicaModeMap:        map[string]types.Mode{},
		Head:                  ProtoLvolToLvol(e.Head),
		Snapshots:             map[string]*Lvol{},
		Frontend:              e.Frontend,
		Endpoint:              e.Endpoint,
		UUID:                  e.Uuid,
		State:                 e.State,
		ErrorMsg:              e.ErrorMsg,
		IsExpanding:           e.IsExpanding,
		LastExpansionError:    e.LastExpansionError,
		LastExpansionFailedAt: e.LastExpansionFailedAt,
	}
	for rName, mode := range e.ReplicaModeMap {
		res.ReplicaModeMap[rName] = types.GRPCReplicaModeToReplicaMode(mode)
	}
	for snapshotName, snapProtoLvol := range e.Snapshots {
		res.Snapshots[snapshotName] = ProtoLvolToLvol(snapProtoLvol)
	}

	return res
}

type EngineFrontend struct {
	Name                  string `json:"name"`
	VolumeName            string `json:"volumeName"`
	EngineName            string `json:"engine_name"`
	SpecSize              uint64 `json:"spec_size"`
	ActualSize            uint64 `json:"actual_size"`
	TargetIP              string `json:"target_ip"`
	TargetPort            int32  `json:"target_port"`
	Frontend              string `json:"frontend"`
	Endpoint              string `json:"endpoint"`
	UUID                  string `json:"uuid"`
	UblkID                int32  `json:"ublk_id"`
	State                 string `json:"state"`
	ErrorMsg              string `json:"error_msg"`
	IsExpanding           bool   `json:"is_expanding"`
	LastExpansionError    string `json:"last_expansion_error"`
	LastExpansionFailedAt string `json:"last_expansion_failed_at"`
}

func ProtoEngineFrontendToEngineFrontend(ef *spdkrpc.EngineFrontend) *EngineFrontend {
	res := &EngineFrontend{
		Name:                  ef.Name,
		VolumeName:            ef.VolumeName,
		EngineName:            ef.EngineName,
		SpecSize:              ef.SpecSize,
		ActualSize:            ef.ActualSize,
		TargetIP:              ef.TargetIp,
		TargetPort:            ef.TargetPort,
		Frontend:              ef.Frontend,
		Endpoint:              ef.Endpoint,
		UUID:                  ef.Uuid,
		UblkID:                ef.UblkId,
		State:                 ef.State,
		ErrorMsg:              ef.ErrorMsg,
		IsExpanding:           ef.IsExpanding,
		LastExpansionError:    ef.LastExpansionError,
		LastExpansionFailedAt: ef.LastExpansionFailedAt,
	}

	return res
}

type EngineFrontendStream struct {
	stream spdkrpc.SPDKService_EngineFrontendWatchClient
}

func NewEngineFrontendStream(stream spdkrpc.SPDKService_EngineFrontendWatchClient) *EngineFrontendStream {
	return &EngineFrontendStream{
		stream,
	}
}

func (s *EngineFrontendStream) Recv() (*emptypb.Empty, error) {
	return s.stream.Recv()
}

type BackingImage struct {
	Name             string `json:"name"`
	BackingImageUUID string `json:"backing_image_uuid"`
	LvsName          string `json:"lvs_name"`
	LvsUUID          string `json:"lvs_uuid"`
	Size             uint64 `json:"size"`
	ExpectedChecksum string `json:"expected_checksum"`
	Snapshot         *Lvol  `json:"snapshot"`
	Progress         int32  `json:"progress"`
	State            string `json:"state"`
	CurrentChecksum  string `json:"current_checksum"`
	ErrorMsg         string `json:"error_msg"`
}

func ProtoBackingImageToBackingImage(bi *spdkrpc.BackingImage) *BackingImage {
	res := &BackingImage{
		Name:             bi.Name,
		BackingImageUUID: bi.BackingImageUuid,
		LvsName:          bi.LvsName,
		LvsUUID:          bi.LvsUuid,
		Size:             bi.Size,
		ExpectedChecksum: bi.ExpectedChecksum,
		Snapshot:         ProtoLvolToLvol(bi.Snapshot),
		Progress:         bi.Progress,
		State:            bi.State,
		CurrentChecksum:  bi.CurrentChecksum,
		ErrorMsg:         bi.ErrorMsg,
	}

	return res
}

func BackingImageToProtoBackingImage(bi *BackingImage) *spdkrpc.BackingImage {
	return &spdkrpc.BackingImage{
		Name:             bi.Name,
		BackingImageUuid: bi.BackingImageUUID,
		LvsName:          bi.LvsName,
		LvsUuid:          bi.LvsUUID,
		Size:             bi.Size,
		ExpectedChecksum: bi.ExpectedChecksum,
		Snapshot:         LvolToProtoLvol(bi.Snapshot),
		Progress:         bi.Progress,
		State:            bi.State,
		CurrentChecksum:  bi.CurrentChecksum,
		ErrorMsg:         bi.ErrorMsg,
	}
}

type DiskInfo struct {
	ID          string
	Name        string
	UUID        string
	Path        string
	Type        string
	TotalSize   int64
	FreeSize    int64
	TotalBlocks int64
	FreeBlocks  int64
	BlockSize   int64
	ClusterSize int64
}

type ReplicaRebuildingStatus struct {
	DstReplicaName    string `json:"dst_replica_name"`
	DstReplicaAddress string `json:"dst_replica_address"`
	SrcReplicaName    string `json:"src_replica_name"`
	SrcReplicaAddress string `json:"src_replica_address"`
	SnapshotName      string `json:"snapshot_name"`
	State             string `json:"state"`
	Progress          uint32 `json:"progress"`
	TotalState        string `json:"total_state"`
	TotalProgress     uint32 `json:"total_progress"`
	Error             string `json:"error"`
}

func ProtoShallowCopyStatusToReplicaRebuildingStatus(replicaName, replicaAddress string, status *spdkrpc.ReplicaRebuildingDstShallowCopyCheckResponse) *ReplicaRebuildingStatus {
	return &ReplicaRebuildingStatus{
		DstReplicaName:    replicaName,
		DstReplicaAddress: replicaAddress,
		SrcReplicaName:    status.SrcReplicaName,
		SrcReplicaAddress: status.SrcReplicaAddress,
		SnapshotName:      status.SnapshotName,
		State:             status.State,
		Progress:          status.Progress,
		TotalState:        status.TotalState,
		TotalProgress:     status.TotalProgress,
		Error:             status.Error,
	}
}

type ReplicaSnapshotCloneSrcStatus struct {
	State             string `json:"state"`
	ProcessedClusters uint64 `json:"processed_clusters"`
	TotalClusters     uint64 `json:"total_clusters"`
	ErrorMsg          string `json:"error_msg"`
}

func ProtoReplicaSnapshotCloneSrcStatusCheckResponseToSnapshotCloneSrcStatus(status *spdkrpc.ReplicaSnapshotCloneSrcStatusCheckResponse) *ReplicaSnapshotCloneSrcStatus {
	state := status.State
	if state == types.SPDKDeepCopyStateInProgress {
		state = types.ProgressStateInProgress
	}
	return &ReplicaSnapshotCloneSrcStatus{
		State:             state,
		ProcessedClusters: status.ProcessedClusters,
		TotalClusters:     status.TotalClusters,
		ErrorMsg:          status.ErrorMsg,
	}
}

type ReplicaSnapshotCloneDstStatus struct {
	IsCloning         bool   `json:"is_cloning"`
	SrcReplicaName    string `json:"src_replica_name"`
	SrcReplicaAddress string `json:"src_replica_address"`
	SnapshotName      string `json:"snapshot_name"`
	State             string `json:"state"`
	Progress          uint32 `json:"progress"`
	Error             string `json:"error"`
}

func ProtoReplicaSnapshotCloneDstStatusCheckResponseToSnapshotCloneDstStatus(status *spdkrpc.ReplicaSnapshotCloneDstStatusCheckResponse) *ReplicaSnapshotCloneDstStatus {
	state := status.State
	if state == types.SPDKDeepCopyStateInProgress {
		state = types.ProgressStateInProgress
	}
	return &ReplicaSnapshotCloneDstStatus{
		IsCloning:         status.IsCloning,
		SrcReplicaName:    status.SrcReplicaName,
		SrcReplicaAddress: status.SrcReplicaAddress,
		SnapshotName:      status.SnapshotName,
		State:             state,
		Progress:          status.Progress,
		Error:             status.Error,
	}
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

type BackingImageStream struct {
	stream spdkrpc.SPDKService_BackingImageWatchClient
}

func NewBackingImageStream(stream spdkrpc.SPDKService_BackingImageWatchClient) *BackingImageStream {
	return &BackingImageStream{
		stream,
	}
}

func (s *BackingImageStream) Recv() (*emptypb.Empty, error) {
	return s.stream.Recv()
}
</file>

<file path="pkg/client/client_backing_image.go">
package client

import (
	"context"
	"fmt"

	"github.com/cockroachdb/errors"
	"google.golang.org/protobuf/types/known/emptypb"

	"github.com/longhorn/types/pkg/generated/spdkrpc"

	"github.com/longhorn/longhorn-spdk-engine/pkg/api"
)

// BackingImageCreate creates a backing image in the specified lvstore.
func (c *SPDKClient) BackingImageCreate(name, backingImageUUID, lvsUUID string, size uint64, checksum string, fromAddress string, srcLvsUUID string) (*api.BackingImage, error) {
	if name == "" || backingImageUUID == "" || checksum == "" || lvsUUID == "" || size == 0 {
		return nil, fmt.Errorf("failed to start SPDK backing image: missing required parameters")
	}
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.BackingImageCreate(ctx, &spdkrpc.BackingImageCreateRequest{
		Name:             name,
		BackingImageUuid: backingImageUUID,
		LvsUuid:          lvsUUID,
		Size:             size,
		Checksum:         checksum,
		FromAddress:      fromAddress,
		SrcLvsUuid:       srcLvsUUID,
	})
	if err != nil {
		return nil, errors.Wrap(err, "failed to start SPDK backing image")
	}
	return api.ProtoBackingImageToBackingImage(resp), nil
}

// BackingImageDelete deletes a backing image from the specified lvstore.
func (c *SPDKClient) BackingImageDelete(name, lvsUUID string) error {
	if name == "" || lvsUUID == "" {
		return fmt.Errorf("failed to delete SPDK backingImage: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.BackingImageDelete(ctx, &spdkrpc.BackingImageDeleteRequest{
		Name:    name,
		LvsUuid: lvsUUID,
	})
	return errors.Wrapf(err, "failed to delete SPDK backing image %v", name)
}

// BackingImageGet returns the current state of a backing image.
func (c *SPDKClient) BackingImageGet(name, lvsUUID string) (*api.BackingImage, error) {
	if name == "" || lvsUUID == "" {
		return nil, fmt.Errorf("failed to get SPDK BackingImage: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.BackingImageGet(ctx, &spdkrpc.BackingImageGetRequest{
		Name:    name,
		LvsUuid: lvsUUID,
	})
	if err != nil {
		return nil, errors.Wrapf(err, "failed to get SPDK backing image %v", name)
	}
	return api.ProtoBackingImageToBackingImage(resp), nil
}

// BackingImageList returns all backing images known to the SPDK service.
func (c *SPDKClient) BackingImageList() (map[string]*api.BackingImage, error) {
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.BackingImageList(ctx, &emptypb.Empty{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to list SPDK backing images")
	}

	res := map[string]*api.BackingImage{}
	for name, backingImage := range resp.BackingImages {
		res[name] = api.ProtoBackingImageToBackingImage(backingImage)
	}
	return res, nil
}

// BackingImageWatch opens a watch stream for backing image change events.
func (c *SPDKClient) BackingImageWatch(ctx context.Context) (*api.BackingImageStream, error) {
	client := c.getSPDKServiceClient()
	stream, err := client.BackingImageWatch(ctx, &emptypb.Empty{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to open backing image watch stream")
	}

	return api.NewBackingImageStream(stream), nil
}

// BackingImageExpose exposes a backing image as a snapshot lvol and returns its address.
func (c *SPDKClient) BackingImageExpose(name, lvsUUID string) (exposedSnapshotLvolAddress string, err error) {
	if name == "" || lvsUUID == "" {
		return "", fmt.Errorf("failed to expose SPDK backing image: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.BackingImageExpose(ctx, &spdkrpc.BackingImageGetRequest{
		Name:    name,
		LvsUuid: lvsUUID,
	})
	if err != nil {
		return "", errors.Wrapf(err, "failed to expose SPDK backing image %v in lvstore: %v", name, lvsUUID)
	}
	return resp.ExposedSnapshotLvolAddress, nil
}

// BackingImageUnexpose stops exposing a backing image in the specified lvstore.
func (c *SPDKClient) BackingImageUnexpose(name, lvsUUID string) error {
	if name == "" || lvsUUID == "" {
		return fmt.Errorf("failed to unexpose SPDK backing image: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.BackingImageUnexpose(ctx, &spdkrpc.BackingImageGetRequest{
		Name:    name,
		LvsUuid: lvsUUID,
	})
	if err != nil {
		return errors.Wrapf(err, "failed to unexpose SPDK backing image %v in lvstore %v", name, lvsUUID)
	}
	return nil
}
</file>

<file path="pkg/client/client_disk.go">
package client

import (
	"context"
	"fmt"

	"github.com/longhorn/types/pkg/generated/spdkrpc"
)

// DiskCreate creates or registers a disk with the given identity and path.
// diskUUID is optional; when empty, the disk is treated as newly added.
func (c *SPDKClient) DiskCreate(diskName, diskUUID, diskPath, diskDriver string, blockSize int64) (*spdkrpc.Disk, error) {
	if diskName == "" || diskPath == "" {
		return nil, fmt.Errorf("failed to create disk: missing required parameters")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	return client.DiskCreate(ctx, &spdkrpc.DiskCreateRequest{
		DiskName:   diskName,
		DiskUuid:   diskUUID,
		DiskPath:   diskPath,
		BlockSize:  blockSize,
		DiskDriver: diskDriver,
	})
}

// DiskGet returns disk information for the specified disk.
func (c *SPDKClient) DiskGet(diskName, diskPath, diskDriver string) (*spdkrpc.Disk, error) {
	if diskName == "" {
		return nil, fmt.Errorf("failed to get disk info: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	return client.DiskGet(ctx, &spdkrpc.DiskGetRequest{
		DiskName:   diskName,
		DiskPath:   diskPath,
		DiskDriver: diskDriver,
	})
}

// DiskDelete removes a disk from the SPDK service.
func (c *SPDKClient) DiskDelete(diskName, diskUUID, diskPath, diskDriver string) error {
	if diskName == "" {
		return fmt.Errorf("failed to delete disk: missing required parameter disk name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.DiskDelete(ctx, &spdkrpc.DiskDeleteRequest{
		DiskName:   diskName,
		DiskUuid:   diskUUID,
		DiskPath:   diskPath,
		DiskDriver: diskDriver,
	})
	return err
}

// DiskHealthGet returns health information for a disk.
func (c *SPDKClient) DiskHealthGet(diskName, diskPath, diskDriver string) (*spdkrpc.DiskHealthGetResponse, error) {
	if diskName == "" {
		return nil, fmt.Errorf("failed to get disk health: missing required parameter 'disk name'")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	req := &spdkrpc.DiskHealthGetRequest{
		DiskName:   diskName,
		DiskDriver: diskDriver,
		DiskPath:   diskPath,
	}
	return client.DiskHealthGet(ctx, req)
}
</file>

<file path="pkg/client/client_engine_frontend.go">
package client

import (
	"context"
	"fmt"

	"github.com/cockroachdb/errors"
	"google.golang.org/protobuf/types/known/emptypb"

	"github.com/longhorn/types/pkg/generated/spdkrpc"

	"github.com/longhorn/longhorn-spdk-engine/pkg/api"
	"github.com/longhorn/longhorn-spdk-engine/pkg/types"
)

// EngineFrontendCreate creates and starts an engine frontend for an existing engine.
func (c *SPDKClient) EngineFrontendCreate(name, volumeName, engineName, frontend string, specSize uint64, targetAddress string,
	ublkQueueDepth, ublkNumberOfQueue int32) (*api.EngineFrontend, error) {
	if name == "" {
		return nil, fmt.Errorf("failed to start engine frontend: missing required parameter name")
	}
	if volumeName == "" {
		return nil, fmt.Errorf("failed to start engine frontend: missing required parameter volumeName")
	}
	if engineName == "" {
		return nil, fmt.Errorf("failed to start engine frontend: missing required parameter engineName")
	}
	if frontend == types.FrontendSPDKTCPBlockdev || frontend == types.FrontendSPDKTCPNvmf {
		if targetAddress == "" {
			return nil, fmt.Errorf("failed to start engine frontend: missing required parameter targetAddress")
		}
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.EngineFrontendCreate(ctx, &spdkrpc.EngineFrontendCreateRequest{
		Name:              name,
		VolumeName:        volumeName,
		EngineName:        engineName,
		SpecSize:          specSize,
		TargetAddress:     targetAddress,
		Frontend:          frontend,
		UblkQueueDepth:    ublkQueueDepth,
		UblkNumberOfQueue: ublkNumberOfQueue,
	})
	if err != nil {
		return nil, errors.Wrap(err, "failed to start engine frontend")
	}

	return api.ProtoEngineFrontendToEngineFrontend(resp), nil
}

// EngineFrontendDelete deletes an engine frontend by name.
func (c *SPDKClient) EngineFrontendDelete(name string) error {
	if name == "" {
		return fmt.Errorf("failed to delete engine frontend: missing required parameter name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineFrontendDelete(ctx, &spdkrpc.EngineFrontendDeleteRequest{
		Name: name,
	})
	return errors.Wrapf(err, "failed to delete engine frontend %v", name)
}

// EngineFrontendList returns all engine frontends known to the SPDK service.
func (c *SPDKClient) EngineFrontendList() (map[string]*api.EngineFrontend, error) {
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.EngineFrontendList(ctx, &emptypb.Empty{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to list engine frontends")
	}

	res := map[string]*api.EngineFrontend{}
	for engineFrontendName, ef := range resp.EngineFrontends {
		res[engineFrontendName] = api.ProtoEngineFrontendToEngineFrontend(ef)
	}
	return res, nil
}

// EngineFrontendWatch opens a watch stream for engine frontend change events.
func (c *SPDKClient) EngineFrontendWatch(ctx context.Context) (*api.EngineFrontendStream, error) {
	client := c.getSPDKServiceClient()
	stream, err := client.EngineFrontendWatch(ctx, &emptypb.Empty{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to open engine frontend watch stream")
	}

	return api.NewEngineFrontendStream(stream), nil
}

// EngineFrontendGet returns the current state of an engine frontend.
func (c *SPDKClient) EngineFrontendGet(name string) (*api.EngineFrontend, error) {
	if name == "" {
		return nil, fmt.Errorf("failed to get engine frontend: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.EngineFrontendGet(ctx, &spdkrpc.EngineFrontendGetRequest{
		Name: name,
	})
	if err != nil {
		return nil, errors.Wrapf(err, "failed to get engine frontend %v", name)
	}
	return api.ProtoEngineFrontendToEngineFrontend(resp), nil
}

// EngineFrontendSwitchOver repoints an engine frontend to a new engine target.
func (c *SPDKClient) EngineFrontendSwitchOver(name, newEngineName, newTargetAddress string) error {
	if name == "" {
		return fmt.Errorf("failed to switch over target for engine frontend: missing required parameter name")
	}
	if newTargetAddress == "" {
		return fmt.Errorf("failed to switch over target for engine frontend: missing required parameter newTargetAddress")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineFrontendSwitchOver(ctx, &spdkrpc.EngineFrontendSwitchOverRequest{
		Name:          name,
		EngineName:    newEngineName,
		TargetAddress: newTargetAddress,
	})
	if err != nil {
		return errors.Wrapf(err, "failed to switch over target to %s with new engine %s for %s", newTargetAddress, newEngineName, name)
	}

	return nil
}

// EngineFrontendSuspend suspends I/O on an engine frontend.
func (c *SPDKClient) EngineFrontendSuspend(name string) error {
	if name == "" {
		return fmt.Errorf("failed to suspend engine frontend: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineFrontendSuspend(ctx, &spdkrpc.EngineFrontendSuspendRequest{
		Name: name,
	})
	if err != nil {
		return errors.Wrapf(err, "failed to suspend engine frontend %v", name)
	}
	return nil
}

// EngineFrontendResume resumes I/O on a suspended engine frontend.
func (c *SPDKClient) EngineFrontendResume(name string) error {
	if name == "" {
		return fmt.Errorf("failed to resume engine frontend: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineFrontendResume(ctx, &spdkrpc.EngineFrontendResumeRequest{
		Name: name,
	})
	if err != nil {
		return errors.Wrapf(err, "failed to resume engine frontend %v", name)
	}
	return nil
}

// EngineFrontendExpand expands an engine frontend and orchestrates the underlying engine expansion.
// The EngineExpand path is for internal orchestration; external callers should use EngineFrontendExpand.
func (c *SPDKClient) EngineFrontendExpand(ctx context.Context, name string, size uint64) error {
	if name == "" {
		return fmt.Errorf("failed to expand engine frontend: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(ctx, GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineFrontendExpand(ctx, &spdkrpc.EngineFrontendExpandRequest{
		Name: name,
		Size: size,
	})
	if err != nil {
		return errors.Wrapf(err, "failed to expand engine frontend %v", name)
	}
	return nil
}

// EngineFrontendSnapshotCreate creates a snapshot through the engine frontend path.
// The EngineSnapshotCreate path is for internal orchestration; external callers should use EngineFrontendSnapshotCreate.
func (c *SPDKClient) EngineFrontendSnapshotCreate(name, snapshotName string) (string, error) {
	if name == "" {
		return "", fmt.Errorf("failed to create snapshot: missing required parameter name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.EngineFrontendSnapshotCreate(ctx, &spdkrpc.SnapshotRequest{
		Name:         name,
		SnapshotName: snapshotName,
	})
	if err != nil {
		return "", errors.Wrapf(err, "failed to create snapshot %s", snapshotName)
	}
	return resp.SnapshotName, nil
}

// EngineFrontendSnapshotDelete deletes a snapshot through the engine frontend path.
// The EngineSnapshotDelete path is for internal orchestration; external callers should use EngineFrontendSnapshotDelete.
func (c *SPDKClient) EngineFrontendSnapshotDelete(name, snapshotName string) error {
	if name == "" || snapshotName == "" {
		return fmt.Errorf("failed to delete engine frontend snapshot: missing required parameter name or snapshot name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineFrontendSnapshotDelete(ctx, &spdkrpc.SnapshotRequest{
		Name:         name,
		SnapshotName: snapshotName,
	})
	return errors.Wrapf(err, "failed to delete engine frontend %s snapshot %s", name, snapshotName)
}

// EngineFrontendSnapshotRevert reverts an engine frontend to the specified snapshot.
func (c *SPDKClient) EngineFrontendSnapshotRevert(name, snapshotName string) error {
	if name == "" || snapshotName == "" {
		return fmt.Errorf("failed to revert engine frontend snapshot: missing required parameter name or snapshot name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineFrontendSnapshotRevert(ctx, &spdkrpc.SnapshotRequest{
		Name:         name,
		SnapshotName: snapshotName,
	})
	return errors.Wrapf(err, "failed to revert engine frontend %s snapshot %s", name, snapshotName)
}

// EngineFrontendSnapshotPurge purges snapshots through the engine frontend path.
// The EngineSnapshotPurge path is for internal orchestration; external callers should use EngineFrontendSnapshotPurge.
func (c *SPDKClient) EngineFrontendSnapshotPurge(name string) error {
	if name == "" {
		return fmt.Errorf("failed to purge engine frontend: missing required parameter name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineFrontendSnapshotPurge(ctx, &spdkrpc.SnapshotRequest{
		Name: name,
	})
	return errors.Wrapf(err, "failed to purge engine frontend %s", name)
}

// EngineFrontendReplicaAdd adds a replica through the engine frontend path.
// The EngineReplicaAdd path is for internal orchestration; external callers should use EngineFrontendReplicaAdd.
func (c *SPDKClient) EngineFrontendReplicaAdd(engineFrontendName, replicaName, replicaAddress string, fastSync bool) error {
	if engineFrontendName == "" {
		return fmt.Errorf("failed to add replica for engine frontend: missing required parameter engineFrontendName")
	}
	if replicaName == "" || replicaAddress == "" {
		return fmt.Errorf("failed to add replica for engine frontend: missing required parameter replicaName or replicaAddress")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineFrontendReplicaAdd(ctx, &spdkrpc.EngineFrontendReplicaAddRequest{
		EngineFrontendName: engineFrontendName,
		ReplicaName:        replicaName,
		ReplicaAddress:     replicaAddress,
		FastSync:           fastSync,
	})
	return errors.Wrapf(err, "failed to add replica %s with address %s by engine frontend %s", replicaName, replicaAddress, engineFrontendName)
}
</file>

<file path="pkg/client/client_engine.go">
package client

import (
	"context"
	"fmt"

	"github.com/cockroachdb/errors"
	"google.golang.org/protobuf/types/known/emptypb"

	"github.com/longhorn/types/pkg/generated/spdkrpc"

	"github.com/longhorn/longhorn-spdk-engine/pkg/api"
	"github.com/longhorn/longhorn-spdk-engine/pkg/util"
)

// EngineCreate creates and starts an engine instance with the requested replicas.
func (c *SPDKClient) EngineCreate(name, volumeName, frontend string, specSize uint64, replicaAddressMap map[string]string, portCount int32, salvageRequested bool) (*api.Engine, error) {
	if name == "" {
		return nil, fmt.Errorf("failed to start engine: missing required parameter name")
	}
	if volumeName == "" {
		return nil, fmt.Errorf("failed to start engine: missing required parameter volumeName")
	}
	if len(replicaAddressMap) == 0 {
		return nil, fmt.Errorf("failed to start engine: missing required parameter replicaAddressMap")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.EngineCreate(ctx, &spdkrpc.EngineCreateRequest{
		Name:              name,
		VolumeName:        volumeName,
		Frontend:          frontend,
		SpecSize:          specSize,
		ReplicaAddressMap: replicaAddressMap,
		PortCount:         portCount,
		SalvageRequested:  salvageRequested,
	})
	if err != nil {
		return nil, errors.Wrap(err, "failed to start engine")
	}

	return api.ProtoEngineToEngine(resp), nil
}

// EngineDelete deletes an engine instance by name.
func (c *SPDKClient) EngineDelete(name string) error {
	if name == "" {
		return fmt.Errorf("failed to delete engine: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineDelete(ctx, &spdkrpc.EngineDeleteRequest{
		Name: name,
	})
	return errors.Wrapf(err, "failed to delete engine %v", name)
}

// EngineGet returns the current state of an engine.
func (c *SPDKClient) EngineGet(name string) (*api.Engine, error) {
	if name == "" {
		return nil, fmt.Errorf("failed to get engine: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.EngineGet(ctx, &spdkrpc.EngineGetRequest{
		Name: name,
	})
	if err != nil {
		return nil, errors.Wrapf(err, "failed to get engine %v", name)
	}
	return api.ProtoEngineToEngine(resp), nil
}

// EngineDeleteTarget deletes the exported target for an engine without deleting the engine object itself.
func (c *SPDKClient) EngineDeleteTarget(name string) error {
	if name == "" {
		return fmt.Errorf("failed to delete target for engine: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineDeleteTarget(ctx, &spdkrpc.EngineDeleteTargetRequest{
		Name: name,
	})
	if err != nil {
		return errors.Wrapf(err, "failed to delete target for engine %v", name)
	}
	return nil
}

// EngineList returns all engines known to the SPDK service.
func (c *SPDKClient) EngineList() (map[string]*api.Engine, error) {
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.EngineList(ctx, &emptypb.Empty{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to list engines")
	}

	res := map[string]*api.Engine{}
	for engineName, e := range resp.Engines {
		res[engineName] = api.ProtoEngineToEngine(e)
	}
	return res, nil
}

// EngineWatch opens a watch stream for engine change events.
func (c *SPDKClient) EngineWatch(ctx context.Context) (*api.EngineStream, error) {
	client := c.getSPDKServiceClient()
	stream, err := client.EngineWatch(ctx, &emptypb.Empty{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to open engine watch stream")
	}

	return api.NewEngineStream(stream), nil
}

// EngineExpand requests an online expansion of the specified engine.
func (c *SPDKClient) EngineExpand(ctx context.Context, name string, size uint64) error {
	if name == "" {
		return fmt.Errorf("failed to expand engine: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(ctx, GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineExpand(ctx, &spdkrpc.EngineExpandRequest{
		Name: name,
		Size: size,
	})
	if err != nil {
		return errors.Wrapf(err, "failed to expand engine %v", name)
	}
	return nil
}

// EngineExpandPrecheck validates whether the specified engine can be expanded to the requested size.
func (c *SPDKClient) EngineExpandPrecheck(ctx context.Context, name string, size uint64) error {
	if name == "" {
		return fmt.Errorf("failed to expand engine precheck: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(ctx, GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineExpandPrecheck(ctx, &spdkrpc.EngineExpandPrecheckRequest{
		Name: name,
		Size: size,
	})
	if err != nil {
		return errors.Wrapf(err, "failed to expand engine %v", name)
	}
	return nil
}

// EngineSnapshotCreate creates a snapshot directly on the engine.
func (c *SPDKClient) EngineSnapshotCreate(name, snapshotName string) (string, error) {
	if name == "" {
		return "", fmt.Errorf("failed to create engine snapshot: missing required parameter name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.EngineSnapshotCreate(ctx, &spdkrpc.SnapshotRequest{
		Name:         name,
		SnapshotName: snapshotName,
	})
	if err != nil {
		return "", errors.Wrapf(err, "failed to create engine %s snapshot %s", name, snapshotName)
	}
	return resp.SnapshotName, nil
}

// EngineSnapshotDelete deletes a snapshot directly from the engine.
func (c *SPDKClient) EngineSnapshotDelete(name, snapshotName string) error {
	if name == "" || snapshotName == "" {
		return fmt.Errorf("failed to delete engine snapshot: missing required parameter name or snapshotName")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineSnapshotDelete(ctx, &spdkrpc.SnapshotRequest{
		Name:         name,
		SnapshotName: snapshotName,
	})
	return errors.Wrapf(err, "failed to delete engine %s snapshot %s", name, snapshotName)
}

// EngineSnapshotRevert reverts an engine directly to the specified snapshot.
func (c *SPDKClient) EngineSnapshotRevert(name, snapshotName string) error {
	if name == "" || snapshotName == "" {
		return fmt.Errorf("failed to revert engine snapshot: missing required parameter name or snapshotName")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineSnapshotRevert(ctx, &spdkrpc.SnapshotRequest{
		Name:         name,
		SnapshotName: snapshotName,
	})
	return errors.Wrapf(err, "failed to revert engine %s snapshot %s", name, snapshotName)
}

// EngineSnapshotPurge purges purgeable snapshots directly on the engine.
func (c *SPDKClient) EngineSnapshotPurge(name string) error {
	if name == "" {
		return fmt.Errorf("failed to purge engine: missing required parameter name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineSnapshotPurge(ctx, &spdkrpc.SnapshotRequest{
		Name: name,
	})
	return errors.Wrapf(err, "failed to purge engine %s", name)
}

// EngineSnapshotHash starts or re-runs checksum generation for an engine snapshot.
func (c *SPDKClient) EngineSnapshotHash(name, snapshotName string, rehash bool) error {
	if name == "" || snapshotName == "" {
		return fmt.Errorf("failed to hash engine snapshot: missing required parameter name or snapshotName")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineSnapshotHash(ctx, &spdkrpc.SnapshotHashRequest{
		Name:         name,
		SnapshotName: snapshotName,
		Rehash:       rehash,
	})
	return errors.Wrapf(err, "failed to hash engine %s snapshot %s", name, snapshotName)
}

// EngineSnapshotHashStatus returns the current checksum status for an engine snapshot.
func (c *SPDKClient) EngineSnapshotHashStatus(name, snapshotName string) (response *spdkrpc.EngineSnapshotHashStatusResponse, err error) {
	if name == "" || snapshotName == "" {
		return nil, fmt.Errorf("failed to check hash status for engine snapshot: missing required parameter name or snapshotName")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	return client.EngineSnapshotHashStatus(ctx, &spdkrpc.SnapshotHashStatusRequest{
		Name:         name,
		SnapshotName: snapshotName,
	})
}

// EngineSnapshotClone clones a snapshot from a source engine into the target engine.
func (c *SPDKClient) EngineSnapshotClone(name, snapshotName, srcEngineName, srcEngineAddress string, cloneMode spdkrpc.CloneMode) error {
	if err := util.VerifyParams(
		util.Param{Name: "name", Value: name},
		util.Param{Name: "snapshotName", Value: snapshotName},
		util.Param{Name: "srcEngineName", Value: srcEngineName},
		util.Param{Name: "srcEngineAddress", Value: srcEngineAddress},
	); err != nil {
		return errors.Wrapf(err, "failed to clone snapshot for engine %s, snapshotName %s, srcEngineName %s, srcEngineAddress %s",
			name, snapshotName, srcEngineName, srcEngineAddress)
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineSnapshotClone(ctx, &spdkrpc.EngineSnapshotCloneRequest{
		Name:             name,
		SnapshotName:     snapshotName,
		SrcEngineName:    srcEngineName,
		SrcEngineAddress: srcEngineAddress,
		CloneMode:        cloneMode,
	})
	return errors.Wrapf(err, "failed to clone snapshot for engine %s, snapshotName %s, srcEngineName %s, srcEngineAddress %s",
		name, snapshotName, srcEngineName, srcEngineAddress)
}

// EngineReplicaAdd calls the full-flow EngineReplicaAdd gRPC on the Engine node.
// When efName and efAddress are non-empty, they are set on the request so
// Engine can call back to the EngineFrontend for suspend/resume.
func (c *SPDKClient) EngineReplicaAdd(engineName, replicaName, replicaAddress string, fastSync bool, efName, efAddress string) error {
	if engineName == "" {
		return fmt.Errorf("failed to add replica for engine: missing required parameter engineName")
	}
	if replicaName == "" || replicaAddress == "" {
		return fmt.Errorf("failed to add replica for engine: missing required parameter replicaName or replicaAddress")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	req := &spdkrpc.EngineReplicaAddRequest{
		EngineName:            engineName,
		ReplicaName:           replicaName,
		ReplicaAddress:        replicaAddress,
		FastSync:              fastSync,
		EngineFrontendName:    efName,
		EngineFrontendAddress: efAddress,
	}

	_, err := client.EngineReplicaAdd(ctx, req)
	return errors.Wrapf(err, "failed to add replica %s with address %s to engine %s", replicaName, replicaAddress, engineName)
}

// EngineReplicaList returns the replicas currently attached to an engine.
func (c *SPDKClient) EngineReplicaList(engineName string) (map[string]*api.Replica, error) {
	if engineName == "" {
		return nil, fmt.Errorf("failed to list replica for engine: missing required parameter engineName")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceLongTimeout)
	defer cancel()

	resp, err := client.EngineReplicaList(ctx, &spdkrpc.EngineReplicaListRequest{
		EngineName: engineName,
	})
	if err != nil {
		return nil, errors.Wrapf(err, "failed to list replica for engine: %s", engineName)
	}
	res := map[string]*api.Replica{}
	for replicaName, r := range resp.Replicas {
		res[replicaName] = api.ProtoReplicaToReplica(r)
	}
	return res, nil
}

// EngineReplicaDelete detaches a replica from an engine.
func (c *SPDKClient) EngineReplicaDelete(engineName, replicaName, replicaAddress string) error {
	if engineName == "" {
		return fmt.Errorf("failed to delete replica from engine: missing required parameter engineName")
	}
	if replicaName == "" && replicaAddress == "" {
		return fmt.Errorf("failed to delete replica from engine: missing required parameter replicaName or replicaAddress, at least one of them is required")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.EngineReplicaDelete(ctx, &spdkrpc.EngineReplicaDeleteRequest{
		EngineName:     engineName,
		ReplicaName:    replicaName,
		ReplicaAddress: replicaAddress,
	})
	return errors.Wrapf(err, "failed to delete replica %s with address %s to engine %s", replicaName, replicaAddress, engineName)
}

// EngineBackupCreate starts a backup from the specified engine snapshot.
func (c *SPDKClient) EngineBackupCreate(req *BackupCreateRequest) (*spdkrpc.BackupCreateResponse, error) {
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	return client.EngineBackupCreate(ctx, &spdkrpc.BackupCreateRequest{
		SnapshotName:         req.SnapshotName,
		BackupTarget:         req.BackupTarget,
		VolumeName:           req.VolumeName,
		EngineName:           req.EngineName,
		Labels:               req.Labels,
		Credential:           req.Credential,
		BackingImageName:     req.BackingImageName,
		BackingImageChecksum: req.BackingImageChecksum,
		BackupName:           req.BackupName,
		CompressionMethod:    req.CompressionMethod,
		ConcurrentLimit:      req.ConcurrentLimit,
		StorageClassName:     req.StorageClassName,
	})
}

// EngineBackupStatus returns the status of an engine backup.
func (c *SPDKClient) EngineBackupStatus(backupName, engineName, replicaAddress string) (*spdkrpc.BackupStatusResponse, error) {
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	return client.EngineBackupStatus(ctx, &spdkrpc.BackupStatusRequest{
		Backup:         backupName,
		EngineName:     engineName,
		ReplicaAddress: replicaAddress,
	})
}

// EngineBackupRestore restores backup data into an engine.
func (c *SPDKClient) EngineBackupRestore(req *BackupRestoreRequest) error {
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	recv, err := client.EngineBackupRestore(ctx, &spdkrpc.EngineBackupRestoreRequest{
		BackupUrl:       req.BackupUrl,
		EngineName:      req.EngineName,
		SnapshotName:    req.SnapshotName,
		Credential:      req.Credential,
		ConcurrentLimit: req.ConcurrentLimit,
	})
	if err != nil {
		return err
	}

	if len(recv.Errors) == 0 {
		return nil
	}

	taskErr := util.NewTaskError()
	for replicaAddress, replicaErr := range recv.Errors {
		replicaURL := "tcp://" + replicaAddress
		taskErr.Append(util.NewReplicaError(replicaURL, errors.New(replicaErr)))
	}

	return taskErr
}

// EngineRestoreStatus returns the current restore status for an engine.
func (c *SPDKClient) EngineRestoreStatus(engineName string) (*spdkrpc.RestoreStatusResponse, error) {
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	return client.EngineRestoreStatus(ctx, &spdkrpc.RestoreStatusRequest{
		EngineName: engineName,
	})
}
</file>

<file path="pkg/client/client_log.go">
package client

import (
	"context"
	"fmt"

	"github.com/cockroachdb/errors"
	"google.golang.org/protobuf/types/known/emptypb"

	"github.com/longhorn/types/pkg/generated/spdkrpc"
)

// LogSetLevel sets the server log level.
func (c *SPDKClient) LogSetLevel(level string) error {
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.LogSetLevel(ctx, &spdkrpc.LogSetLevelRequest{
		Level: level,
	})
	return err
}

// LogSetFlags enables the specified server log flags.
func (c *SPDKClient) LogSetFlags(flags string) error {
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.LogSetFlags(ctx, &spdkrpc.LogSetFlagsRequest{
		Flags: flags,
	})
	return err
}

// LogGetLevel returns the current server log level.
func (c *SPDKClient) LogGetLevel() (string, error) {
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.LogGetLevel(ctx, &emptypb.Empty{})
	if err != nil {
		return "", err
	}
	return resp.Level, nil
}

// LogGetFlags returns the currently enabled server log flags.
func (c *SPDKClient) LogGetFlags() (string, error) {
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.LogGetFlags(ctx, &emptypb.Empty{})
	if err != nil {
		return "", err
	}
	return resp.Flags, nil
}

// MetricsGet returns metrics for the specified object.
func (c *SPDKClient) MetricsGet(name string) (*spdkrpc.Metrics, error) {
	if name == "" {
		return nil, fmt.Errorf("failed to get engine metrics: missing required parameter")
	}
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()
	resp, err := client.MetricsGet(ctx, &spdkrpc.MetricsRequest{
		Name: name,
	})
	if err != nil {
		return nil, errors.Wrapf(err, "failed to get engine %v metrics", name)
	}
	return resp, nil
}
</file>

<file path="pkg/client/client_replica.go">
package client

import (
	"context"
	"fmt"

	"github.com/cockroachdb/errors"
	"google.golang.org/protobuf/types/known/emptypb"

	"github.com/longhorn/types/pkg/generated/spdkrpc"

	"github.com/longhorn/longhorn-spdk-engine/pkg/api"
	"github.com/longhorn/longhorn-spdk-engine/pkg/util"
)

// ReplicaCreate creates and starts a replica in the specified lvstore.
func (c *SPDKClient) ReplicaCreate(name, lvsName, lvsUUID string, specSize uint64, portCount int32, backingImageName string) (*api.Replica, error) {
	if name == "" || lvsName == "" || lvsUUID == "" {
		return nil, fmt.Errorf("failed to start SPDK replica: missing required parameters")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.ReplicaCreate(ctx, &spdkrpc.ReplicaCreateRequest{
		Name:             name,
		LvsName:          lvsName,
		LvsUuid:          lvsUUID,
		SpecSize:         specSize,
		PortCount:        portCount,
		BackingImageName: backingImageName,
	})
	if err != nil {
		return nil, errors.Wrap(err, "failed to start SPDK replica")
	}

	return api.ProtoReplicaToReplica(resp), nil
}

// ReplicaDelete deletes a replica, optionally cleaning up related SPDK resources.
func (c *SPDKClient) ReplicaDelete(name string, cleanupRequired bool) error {
	if name == "" {
		return fmt.Errorf("failed to delete SPDK replica: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.ReplicaDelete(ctx, &spdkrpc.ReplicaDeleteRequest{
		Name:            name,
		CleanupRequired: cleanupRequired,
	})
	return errors.Wrapf(err, "failed to delete SPDK replica %v", name)
}

// ReplicaGet returns the current state of a replica.
func (c *SPDKClient) ReplicaGet(name string) (*api.Replica, error) {
	if name == "" {
		return nil, fmt.Errorf("failed to get SPDK replica: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.ReplicaGet(ctx, &spdkrpc.ReplicaGetRequest{
		Name: name,
	})
	if err != nil {
		return nil, errors.Wrapf(err, "failed to get SPDK replica %v", name)
	}
	return api.ProtoReplicaToReplica(resp), nil
}

// ReplicaExpand requests an online expansion of the specified replica.
func (c *SPDKClient) ReplicaExpand(name string, size uint64) error {
	if name == "" {
		return fmt.Errorf("failed to expand replica: missing required parameter")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.ReplicaExpand(ctx, &spdkrpc.ReplicaExpandRequest{
		Name: name,
		Size: size,
	})
	return errors.Wrapf(err, "failed to expand replica %v", name)
}

// ReplicaList returns all replicas known to the SPDK service.
func (c *SPDKClient) ReplicaList() (map[string]*api.Replica, error) {
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.ReplicaList(ctx, &emptypb.Empty{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to list SPDK replicas")
	}

	res := map[string]*api.Replica{}
	for replicaName, r := range resp.Replicas {
		res[replicaName] = api.ProtoReplicaToReplica(r)
	}
	return res, nil
}

// ReplicaWatch opens a watch stream for replica change events.
func (c *SPDKClient) ReplicaWatch(ctx context.Context) (*api.ReplicaStream, error) {
	client := c.getSPDKServiceClient()
	stream, err := client.ReplicaWatch(ctx, &emptypb.Empty{})
	if err != nil {
		return nil, errors.Wrap(err, "failed to open replica watch stream")
	}

	return api.NewReplicaStream(stream), nil
}

// ReplicaSnapshotCreate creates a snapshot directly on a replica.
func (c *SPDKClient) ReplicaSnapshotCreate(name, snapshotName string, opts *api.SnapshotOptions) error {
	if name == "" || snapshotName == "" || opts == nil {
		return fmt.Errorf("failed to create SPDK replica snapshot: missing required parameter name, snapshot name or opts")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	snapshotRequest := spdkrpc.SnapshotRequest{
		Name:              name,
		SnapshotName:      snapshotName,
		UserCreated:       opts.UserCreated,
		SnapshotTimestamp: opts.Timestamp,
	}

	_, err := client.ReplicaSnapshotCreate(ctx, &snapshotRequest)

	return errors.Wrapf(err, "failed to create SPDK replica %s snapshot %s", name, snapshotName)
}

// ReplicaSnapshotDelete deletes a snapshot directly from a replica.
func (c *SPDKClient) ReplicaSnapshotDelete(name, snapshotName string) error {
	if name == "" || snapshotName == "" {
		return fmt.Errorf("failed to delete SPDK replica snapshot: missing required parameter name or snapshot name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.ReplicaSnapshotDelete(ctx, &spdkrpc.SnapshotRequest{
		Name:         name,
		SnapshotName: snapshotName,
	})
	return errors.Wrapf(err, "failed to delete SPDK replica %s snapshot %s", name, snapshotName)
}

// ReplicaSnapshotRevert reverts a replica directly to the specified snapshot.
func (c *SPDKClient) ReplicaSnapshotRevert(name, snapshotName string) error {
	if name == "" || snapshotName == "" {
		return fmt.Errorf("failed to revert SPDK replica snapshot: missing required parameter name or snapshot name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.ReplicaSnapshotRevert(ctx, &spdkrpc.SnapshotRequest{
		Name:         name,
		SnapshotName: snapshotName,
	})
	return errors.Wrapf(err, "failed to revert SPDK replica %s snapshot %s", name, snapshotName)
}

// ReplicaSnapshotPurge purges purgeable snapshots directly on a replica.
func (c *SPDKClient) ReplicaSnapshotPurge(name string) error {
	if name == "" {
		return fmt.Errorf("failed to purge SPDK replica: missing required parameter name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.ReplicaSnapshotPurge(ctx, &spdkrpc.SnapshotRequest{
		Name: name,
	})
	return errors.Wrapf(err, "failed to purge SPDK replica %s", name)
}

// ReplicaSnapshotHash starts or re-runs checksum generation for a replica snapshot.
func (c *SPDKClient) ReplicaSnapshotHash(name, snapshotName string, rehash bool) error {
	if name == "" || snapshotName == "" {
		return fmt.Errorf("failed to hash SPDK replica snapshot: missing required parameter name or snapshot name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.ReplicaSnapshotHash(ctx, &spdkrpc.SnapshotHashRequest{
		Name:         name,
		SnapshotName: snapshotName,
		Rehash:       rehash,
	})
	return errors.Wrapf(err, "failed to hash SPDK replica %s snapshot %s", name, snapshotName)
}

// ReplicaSnapshotHashStatus returns the current checksum status for a replica snapshot.
func (c *SPDKClient) ReplicaSnapshotHashStatus(name, snapshotName string) (*spdkrpc.ReplicaSnapshotHashStatusResponse, error) {
	if name == "" || snapshotName == "" {
		return nil, fmt.Errorf("failed to check hash status for SPDK replica snapshot: missing required parameter name or snapshot name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	return client.ReplicaSnapshotHashStatus(ctx, &spdkrpc.SnapshotHashStatusRequest{
		Name:         name,
		SnapshotName: snapshotName,
	})
}

// ReplicaSnapshotCloneDstStart starts snapshot clone preparation on the destination replica.
func (c *SPDKClient) ReplicaSnapshotCloneDstStart(name, snapshotName, srcReplicaName, srcReplicaAddress string, cloneMode spdkrpc.CloneMode) (err error) {
	defer func() {
		err = errors.Wrapf(err, "failed to do ReplicaSnapshotCloneDstStart: replica: %v, snapshot: %v", name, snapshotName)
	}()
	if err := util.VerifyParams(
		util.Param{Name: "name", Value: name},
		util.Param{Name: "snapshotName", Value: snapshotName},
		util.Param{Name: "srcReplicaName", Value: srcReplicaName},
		util.Param{Name: "srcReplicaAddress", Value: srcReplicaAddress},
	); err != nil {
		return err
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err = client.ReplicaSnapshotCloneDstStart(ctx, &spdkrpc.ReplicaSnapshotCloneDstStartRequest{
		Name:              name,
		SnapshotName:      snapshotName,
		SrcReplicaName:    srcReplicaName,
		SrcReplicaAddress: srcReplicaAddress,
		CloneMode:         cloneMode,
	})
	return err
}

// ReplicaSnapshotCloneDstStatusCheck returns snapshot clone progress on the destination replica.
func (c *SPDKClient) ReplicaSnapshotCloneDstStatusCheck(name string) (resp *api.ReplicaSnapshotCloneDstStatus, err error) {
	defer func() {
		err = errors.Wrapf(err, "failed to do ReplicaSnapshotCloneDstStatusCheck: replica name %v", name)
	}()
	if err := util.VerifyParams(
		util.Param{Name: "name", Value: name},
	); err != nil {
		return nil, err
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	rpcResp, err := client.ReplicaSnapshotCloneDstStatusCheck(ctx, &spdkrpc.ReplicaSnapshotCloneDstStatusCheckRequest{
		Name: name,
	})
	if err != nil {
		return nil, err
	}
	return api.ProtoReplicaSnapshotCloneDstStatusCheckResponseToSnapshotCloneDstStatus(rpcResp), nil
}

// ReplicaSnapshotCloneSrcStart starts snapshot clone work on the source replica.
func (c *SPDKClient) ReplicaSnapshotCloneSrcStart(name, snapshotName, dstReplicaName, dstCloningLvolAddress string, mode spdkrpc.CloneMode) (err error) {
	defer func() {
		err = errors.Wrapf(err, "failed to do ReplicaSnapshotCloneSrcStart. Replica name: %v, snapshot name: %v", name, snapshotName)
	}()
	if err := util.VerifyParams(
		util.Param{Name: "name", Value: name},
		util.Param{Name: "snapshotName", Value: snapshotName},
		util.Param{Name: "dstReplicaName", Value: dstReplicaName},
	); err != nil {
		return err
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err = client.ReplicaSnapshotCloneSrcStart(ctx, &spdkrpc.ReplicaSnapshotCloneSrcStartRequest{
		Name:                  name,
		SnapshotName:          snapshotName,
		DstReplicaName:        dstReplicaName,
		DstCloningLvolAddress: dstCloningLvolAddress,
		CloneMode:             mode,
	})
	return err
}

// ReplicaSnapshotCloneSrcStatusCheck returns snapshot clone progress on the source replica.
func (c *SPDKClient) ReplicaSnapshotCloneSrcStatusCheck(name, snapshotName, dstReplicaName string) (resp *api.ReplicaSnapshotCloneSrcStatus, err error) {
	defer func() {
		err = errors.Wrapf(err, "failed to do ReplicaSnapshotCloneSrcStatusCheck. Replica name: %v, snapshot name: %v", name, snapshotName)
	}()
	if err := util.VerifyParams(
		util.Param{Name: "name", Value: name},
		util.Param{Name: "snapshotName", Value: snapshotName},
		util.Param{Name: "dstReplicaName", Value: dstReplicaName},
	); err != nil {
		return nil, err
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	rpcResp, err := client.ReplicaSnapshotCloneSrcStatusCheck(ctx, &spdkrpc.ReplicaSnapshotCloneSrcStatusCheckRequest{
		Name:           name,
		SnapshotName:   snapshotName,
		DstReplicaName: dstReplicaName,
	})
	if err != nil {
		return nil, err
	}
	return api.ProtoReplicaSnapshotCloneSrcStatusCheckResponseToSnapshotCloneSrcStatus(rpcResp), nil
}

// ReplicaSnapshotCloneSrcFinish finalizes source-side snapshot clone state and cleans up clone metadata.
func (c *SPDKClient) ReplicaSnapshotCloneSrcFinish(name, dstReplicaName string) (err error) {
	defer func() {
		err = errors.Wrapf(err, "failed to do ReplicaSnapshotCloneSrcFinish. replica: %v, src replica: %v", dstReplicaName, name)
	}()
	if err := util.VerifyParams(
		util.Param{Name: "name", Value: name},
		util.Param{Name: "dstReplicaName", Value: dstReplicaName},
	); err != nil {
		return err
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err = client.ReplicaSnapshotCloneSrcFinish(ctx, &spdkrpc.ReplicaSnapshotCloneSrcFinishRequest{
		Name:           name,
		DstReplicaName: dstReplicaName,
	})
	return err
}

// ReplicaSnapshotRangeHashGet returns range hashes for the specified clusters of a replica snapshot.
func (c *SPDKClient) ReplicaSnapshotRangeHashGet(name, snapshotName string, clusterStartIndex, clusterCount uint64) (*spdkrpc.ReplicaSnapshotRangeHashGetResponse, error) {
	if name == "" || snapshotName == "" {
		return nil, fmt.Errorf("failed to get range hash for SPDK replica snapshot: missing required parameter name or snapshot name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	return client.ReplicaSnapshotRangeHashGet(ctx, &spdkrpc.ReplicaSnapshotRangeHashGetRequest{
		Name:              name,
		SnapshotName:      snapshotName,
		ClusterStartIndex: clusterStartIndex,
		ClusterCount:      clusterCount,
	})
}

// ReplicaRebuildingSrcStart asks the source replica to check the parent snapshot of the head and expose it as a NVMf bdev if necessary.
// If the source replica and the destination replica have different IPs, the API will expose the snapshot lvol as a NVMf bdev and return the address <IP>:<Port>.
// Otherwise, the API will directly return the snapshot lvol alias.
func (c *SPDKClient) ReplicaRebuildingSrcStart(srcReplicaName, dstReplicaName, dstReplicaAddress, exposedSnapshotName string) (exposedSnapshotLvolAddress string, err error) {
	if srcReplicaName == "" {
		return "", fmt.Errorf("failed to start replica rebuilding src: missing required parameter src replica name")
	}
	if dstReplicaName == "" || dstReplicaAddress == "" {
		return "", fmt.Errorf("failed to start replica rebuilding src: missing required parameter dst replica name or address")
	}
	if exposedSnapshotName == "" {
		return "", fmt.Errorf("failed to start replica rebuilding src: missing required parameter exposed snapshot name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	resp, err := client.ReplicaRebuildingSrcStart(ctx, &spdkrpc.ReplicaRebuildingSrcStartRequest{
		Name:                srcReplicaName,
		DstReplicaName:      dstReplicaName,
		DstReplicaAddress:   dstReplicaAddress,
		ExposedSnapshotName: exposedSnapshotName,
	})
	if err != nil {
		return "", errors.Wrapf(err, "failed to start replica rebuilding src %s for rebuilding replica %s(%s)", srcReplicaName, dstReplicaName, dstReplicaAddress)
	}
	return resp.ExposedSnapshotLvolAddress, nil
}

// ReplicaRebuildingSrcFinish asks the source replica to stop exposing the parent snapshot of the head, if needed,
// and to clean up destination-replica rebuild state. It does not detach the destination rebuilding lvol.
func (c *SPDKClient) ReplicaRebuildingSrcFinish(srcReplicaName, dstReplicaName string) error {
	if srcReplicaName == "" {
		return fmt.Errorf("failed to finish replica rebuilding src: missing required parameter src replica name")
	}
	if dstReplicaName == "" {
		return fmt.Errorf("failed to finish replica rebuilding src: missing required parameter dst replica name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.ReplicaRebuildingSrcFinish(ctx, &spdkrpc.ReplicaRebuildingSrcFinishRequest{
		Name:           srcReplicaName,
		DstReplicaName: dstReplicaName,
	})
	return errors.Wrapf(err, "failed to finish replica rebuilding src %s for rebuilding replica %s", srcReplicaName, dstReplicaName)
}

// ReplicaRebuildingSrcShallowCopyStart starts a shallow copy from the source snapshot lvol to the destination rebuilding lvol.
func (c *SPDKClient) ReplicaRebuildingSrcShallowCopyStart(srcReplicaName, snapshotName, dstRebuildingLvolAddress string) error {
	if srcReplicaName == "" || snapshotName == "" {
		return fmt.Errorf("failed to start rebuilding src replica shallow copy: missing required parameter replica name or snapshot name")
	}
	if dstRebuildingLvolAddress == "" {
		return fmt.Errorf("failed to start rebuilding src replica shallow copy: missing required parameter dst rebuilding lvol address")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceMedTimeout)
	defer cancel()

	_, err := client.ReplicaRebuildingSrcShallowCopyStart(ctx, &spdkrpc.ReplicaRebuildingSrcShallowCopyStartRequest{
		Name:                     srcReplicaName,
		SnapshotName:             snapshotName,
		DstRebuildingLvolAddress: dstRebuildingLvolAddress,
	})
	if err != nil {
		return errors.Wrapf(err, "failed to start rebuilding src replica %v shallow copy snapshot %v", srcReplicaName, snapshotName)
	}
	return nil
}

// ReplicaRebuildingSrcRangeShallowCopyStart starts a delta shallow copy for the specified source clusters.
func (c *SPDKClient) ReplicaRebuildingSrcRangeShallowCopyStart(srcReplicaName, snapshotName, dstRebuildingLvolAddress string, mismatchingClusterList []uint64) error {
	if srcReplicaName == "" || snapshotName == "" {
		return fmt.Errorf("failed to start rebuilding src replica range shallow copy: missing required parameter replica name or snapshot name")
	}
	if dstRebuildingLvolAddress == "" {
		return fmt.Errorf("failed to start rebuilding src replica range shallow copy: missing required parameter dst rebuilding lvol address")
	}
	if len(mismatchingClusterList) == 0 {
		return fmt.Errorf("failed to start rebuilding src replica range shallow copy: missing required parameter mismatching cluster list")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceMedTimeout)
	defer cancel()

	_, err := client.ReplicaRebuildingSrcRangeShallowCopyStart(ctx, &spdkrpc.ReplicaRebuildingSrcRangeShallowCopyStartRequest{
		Name:                     srcReplicaName,
		SnapshotName:             snapshotName,
		DstRebuildingLvolAddress: dstRebuildingLvolAddress,
		MismatchingClusterList:   mismatchingClusterList,
	})
	if err != nil {
		return errors.Wrapf(err, "failed to start rebuilding src replica %v range shallow copy snapshot %v", srcReplicaName, snapshotName)
	}
	return nil
}

// ReplicaRebuildingSrcShallowCopyCheck returns source-side shallow copy progress for a rebuilding snapshot.
func (c *SPDKClient) ReplicaRebuildingSrcShallowCopyCheck(srcReplicaName, dstReplicaName, snapshotName string) (state string, handledClusters, totalClusters uint64, errorMsg string, err error) {
	if srcReplicaName == "" || dstReplicaName == "" {
		return "", 0, 0, "", fmt.Errorf("failed to check rebuilding src replica shallow copy: missing required parameter src replica name or dst replica name")
	}
	if snapshotName == "" {
		return "", 0, 0, "", fmt.Errorf("failed to check rebuilding src replica shallow copy: missing required parameter snapshot name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceMedTimeout)
	defer cancel()

	resp, err := client.ReplicaRebuildingSrcShallowCopyCheck(ctx, &spdkrpc.ReplicaRebuildingSrcShallowCopyCheckRequest{
		Name:           srcReplicaName,
		DstReplicaName: dstReplicaName,
		SnapshotName:   snapshotName,
	})
	if err != nil {
		return "", 0, 0, "", errors.Wrapf(err, "failed to check rebuilding src replica %v shallow copy snapshot %v for dst replica %s", srcReplicaName, snapshotName, dstReplicaName)
	}
	return resp.State, resp.HandledClusters, resp.TotalClusters, resp.ErrorMsg, nil
}

// ReplicaRebuildingDstStart prepares the destination replica for rebuilding from a source snapshot.
// It creates a new head lvol, exposes it as needed, and returns the destination head lvol address.
// The external snapshot address is a local alias when source and destination share the same host,
// otherwise it is the exported NVMf address of the source snapshot lvol.
func (c *SPDKClient) ReplicaRebuildingDstStart(replicaName, srcReplicaName, srcReplicaAddress, externalSnapshotName, externalSnapshotAddress string, rebuildingSnapshotList []*api.Lvol) (dstHeadLvolAddress string, err error) {
	if replicaName == "" {
		return "", fmt.Errorf("failed to start replica rebuilding dst: missing required parameter replica name")
	}
	if srcReplicaName == "" || srcReplicaAddress == "" {
		return "", fmt.Errorf("failed to start replica rebuilding dst: missing required parameter src replica name or address")
	}
	if externalSnapshotName == "" || externalSnapshotAddress == "" {
		return "", fmt.Errorf("failed to start replica rebuilding dst: missing required parameter external snapshot name or address")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	var protoRebuildingSnapshotList []*spdkrpc.Lvol
	for _, snapshot := range rebuildingSnapshotList {
		protoRebuildingSnapshotList = append(protoRebuildingSnapshotList, api.LvolToProtoLvol(snapshot))
	}
	resp, err := client.ReplicaRebuildingDstStart(ctx, &spdkrpc.ReplicaRebuildingDstStartRequest{
		Name:                    replicaName,
		SrcReplicaName:          srcReplicaName,
		SrcReplicaAddress:       srcReplicaAddress,
		ExternalSnapshotName:    externalSnapshotName,
		ExternalSnapshotAddress: externalSnapshotAddress,
		RebuildingSnapshotList:  protoRebuildingSnapshotList,
	})
	if err != nil {
		return "", errors.Wrapf(err, "failed to start replica rebuilding dst %s", replicaName)
	}
	return resp.DstHeadLvolAddress, nil
}

// ReplicaRebuildingDstFinish finalizes rebuild state on the destination replica.
// It reconstructs the snapshot tree and active chain, then detaches the external source snapshot if needed.
// The caller must guarantee that there is no I/O during the parent switch.
func (c *SPDKClient) ReplicaRebuildingDstFinish(replicaName string) error {
	if replicaName == "" {
		return fmt.Errorf("failed to finish replica rebuilding dst: missing required parameter replica name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.ReplicaRebuildingDstFinish(ctx, &spdkrpc.ReplicaRebuildingDstFinishRequest{
		Name: replicaName,
	})
	return errors.Wrapf(err, "failed to finish replica rebuilding dst %s", replicaName)
}

// ReplicaRebuildingDstShallowCopyStart starts shallow copy work on the destination replica.
func (c *SPDKClient) ReplicaRebuildingDstShallowCopyStart(dstReplicaName, snapshotName string, fastSync bool) error {
	if dstReplicaName == "" {
		return fmt.Errorf("failed to start rebuilding dst replica shallow copy: missing required parameter dst replica name")
	}
	if snapshotName == "" {
		return fmt.Errorf("failed to start rebuilding dst replica shallow copy: missing required parameter snapshot name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceMedTimeout)
	defer cancel()

	_, err := client.ReplicaRebuildingDstShallowCopyStart(ctx, &spdkrpc.ReplicaRebuildingDstShallowCopyStartRequest{
		Name:         dstReplicaName,
		SnapshotName: snapshotName,
		FastSync:     fastSync,
	})
	return errors.Wrapf(err, "failed to start rebuilding dst replica %v shallow copy snapshot %v", dstReplicaName, snapshotName)
}

// ReplicaRebuildingDstShallowCopyCheck returns destination-side shallow copy progress.
func (c *SPDKClient) ReplicaRebuildingDstShallowCopyCheck(dstReplicaName string) (resp *api.ReplicaRebuildingStatus, err error) {
	if dstReplicaName == "" {
		return nil, fmt.Errorf("failed to check rebuilding dst replica shallow copy: missing required parameter dst replica name")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceMedTimeout)
	defer cancel()

	rpcResp, err := client.ReplicaRebuildingDstShallowCopyCheck(ctx, &spdkrpc.ReplicaRebuildingDstShallowCopyCheckRequest{
		Name: dstReplicaName,
	})
	if err != nil {
		return nil, errors.Wrapf(err, "failed to check rebuilding dst replica %v shallow copy snapshot", dstReplicaName)
	}
	return api.ProtoShallowCopyStatusToReplicaRebuildingStatus(dstReplicaName, c.serviceURL, rpcResp), nil
}

// ReplicaRebuildingDstSnapshotCreate creates a rebuilding snapshot on the destination replica.
func (c *SPDKClient) ReplicaRebuildingDstSnapshotCreate(name, snapshotName string, opts *api.SnapshotOptions) error {
	if name == "" || snapshotName == "" || opts == nil {
		return fmt.Errorf("failed to create dst SPDK replica rebuilding snapshot: missing required parameter name, snapshot name or opts")
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	snapshotRequest := spdkrpc.SnapshotRequest{
		Name:              name,
		SnapshotName:      snapshotName,
		UserCreated:       opts.UserCreated,
		SnapshotTimestamp: opts.Timestamp,
	}

	_, err := client.ReplicaRebuildingDstSnapshotCreate(ctx, &snapshotRequest)
	return errors.Wrapf(err, "failed to create dst SPDK replica %s rebuilding snapshot %s", name, snapshotName)
}

// ReplicaRebuildingDstSetQosLimit sets a QoS limit (in MB/s) on the destination replica
// during the shallow copy (rebuilding) process. The limit controls write throughput to reduce rebuild impact.
// A QoS limit of 0 disables throttling (i.e., unlimited bandwidth).
func (c *SPDKClient) ReplicaRebuildingDstSetQosLimit(replicaName string, qosLimitMbps int64) error {
	if replicaName == "" {
		return fmt.Errorf("failed to set QoS on replica: missing replica name")
	}
	if qosLimitMbps < 0 {
		return fmt.Errorf("invalid QoS limit: must not be negative, got %d", qosLimitMbps)
	}

	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceMedTimeout)
	defer cancel()

	_, err := client.ReplicaRebuildingDstSetQosLimit(ctx, &spdkrpc.ReplicaRebuildingDstSetQosLimitRequest{
		Name:         replicaName,
		QosLimitMbps: qosLimitMbps,
	})
	if err != nil {
		return errors.Wrapf(err, "failed to set QoS limit %d MB/s on replica %s", qosLimitMbps, replicaName)
	}

	return nil
}

// ReplicaBackupCreate starts a backup from the specified replica snapshot.
func (c *SPDKClient) ReplicaBackupCreate(req *BackupCreateRequest) (*spdkrpc.BackupCreateResponse, error) {
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	return client.ReplicaBackupCreate(ctx, &spdkrpc.BackupCreateRequest{
		BackupName:           req.BackupName,
		SnapshotName:         req.SnapshotName,
		BackupTarget:         req.BackupTarget,
		VolumeName:           req.VolumeName,
		ReplicaName:          req.ReplicaName,
		Size:                 int64(req.Size),
		Labels:               req.Labels,
		Credential:           req.Credential,
		BackingImageName:     req.BackingImageName,
		BackingImageChecksum: req.BackingImageChecksum,
		CompressionMethod:    req.CompressionMethod,
		ConcurrentLimit:      req.ConcurrentLimit,
		StorageClassName:     req.StorageClassName,
	})
}

// ReplicaBackupStatus returns the status of a replica backup.
func (c *SPDKClient) ReplicaBackupStatus(backupName string) (*spdkrpc.BackupStatusResponse, error) {
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	return client.ReplicaBackupStatus(ctx, &spdkrpc.BackupStatusRequest{
		Backup: backupName,
	})
}

// ReplicaBackupRestore restores backup data into a replica.
func (c *SPDKClient) ReplicaBackupRestore(req *BackupRestoreRequest) error {
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	_, err := client.ReplicaBackupRestore(ctx, &spdkrpc.ReplicaBackupRestoreRequest{
		BackupUrl:       req.BackupUrl,
		ReplicaName:     req.ReplicaName,
		SnapshotName:    req.SnapshotName,
		Credential:      req.Credential,
		ConcurrentLimit: req.ConcurrentLimit,
	})
	return err
}

// ReplicaRestoreStatus returns the current restore status for a replica.
func (c *SPDKClient) ReplicaRestoreStatus(replicaName string) (*spdkrpc.ReplicaRestoreStatusResponse, error) {
	client := c.getSPDKServiceClient()
	ctx, cancel := context.WithTimeout(context.Background(), GRPCServiceTimeout)
	defer cancel()

	return client.ReplicaRestoreStatus(ctx, &spdkrpc.ReplicaRestoreStatusRequest{
		ReplicaName: replicaName,
	})
}
</file>

<file path="pkg/client/client.go">
package client

import (
	"github.com/cockroachdb/errors"
	"google.golang.org/grpc"

	"github.com/longhorn/types/pkg/generated/spdkrpc"
	"google.golang.org/grpc/credentials/insecure"
)

// Close closes the underlying gRPC connection for the SPDK service context.
func (c *SPDKServiceContext) Close() error {
	if c.cc != nil {
		if err := c.cc.Close(); err != nil {
			return err
		}
		c.cc = nil
	}
	return nil
}

func (c *SPDKClient) getSPDKServiceClient() spdkrpc.SPDKServiceClient {
	return c.service
}

// NewSPDKClient creates an SPDK gRPC client connected to the given service URL.
func NewSPDKClient(serviceURL string) (*SPDKClient, error) {
	getSPDKServiceContext := func(serviceUrl string) (SPDKServiceContext, error) {
		// Disable gRPC service config discovery to prevent DNS flooding in Kubernetes
		connection, err := grpc.NewClient(
			serviceUrl,
			grpc.WithTransportCredentials(insecure.NewCredentials()),
			grpc.WithNoProxy(),
			grpc.WithDisableServiceConfig(),
		)
		if err != nil {
			return SPDKServiceContext{}, errors.Wrapf(err, "cannot connect to SPDKService %v", serviceUrl)
		}

		return SPDKServiceContext{
			cc:      connection,
			service: spdkrpc.NewSPDKServiceClient(connection),
		}, nil
	}

	serviceContext, err := getSPDKServiceContext(serviceURL)
	if err != nil {
		return nil, err
	}

	return &SPDKClient{
		serviceURL:         serviceURL,
		SPDKServiceContext: serviceContext,
	}, nil
}
</file>

<file path="pkg/client/types.go">
package client

import (
	"time"

	"google.golang.org/grpc"

	"github.com/longhorn/types/pkg/generated/spdkrpc"
)

const (
	GRPCServiceTimeout     = 3 * time.Minute
	GRPCServiceMedTimeout  = 24 * time.Hour
	GRPCServiceLongTimeout = 72 * time.Hour
)

type SPDKServiceContext struct {
	cc      *grpc.ClientConn
	service spdkrpc.SPDKServiceClient
}

type SPDKClient struct {
	serviceURL string
	SPDKServiceContext
}

type BackupCreateRequest struct {
	BackupName           string
	SnapshotName         string
	VolumeName           string
	EngineName           string
	ReplicaName          string
	Size                 uint64
	BackupTarget         string
	StorageClassName     string
	BackingImageName     string
	BackingImageChecksum string
	CompressionMethod    string
	ConcurrentLimit      int32
	Labels               []string
	Credential           map[string]string
}

type BackupRestoreRequest struct {
	BackupUrl       string
	EngineName      string
	ReplicaName     string
	SnapshotName    string
	Credential      map[string]string
	ConcurrentLimit int32
}
</file>

<file path="pkg/log/log.go">
package log

import (
	"reflect"
	"sync"

	"github.com/jinzhu/copier"
	"github.com/sirupsen/logrus"
)

type SafeLogger struct {
	sync.RWMutex

	logger logrus.Ext1FieldLogger
}

// NewSafeLogger creates a new thread-safe logger instance
func NewSafeLogger(logger logrus.Ext1FieldLogger) *SafeLogger {
	if logger == nil {
		logger = logrus.StandardLogger()
	}
	return &SafeLogger{
		logger: logger,
	}
}

// WithError adds an error field to the logger thread-safely
func (s *SafeLogger) WithError(err error) logrus.Ext1FieldLogger {
	s.RLock()
	defer s.RUnlock()
	return s.logger.WithError(err)
}

// Trace logs a trace message thread-safely
func (s *SafeLogger) Trace(args ...interface{}) {
	s.Lock()
	defer s.Unlock()
	s.logger.Trace(args...)
}

// Tracef logs a formatted trace message thread-safely
func (s *SafeLogger) Tracef(format string, args ...interface{}) {
	s.Lock()
	defer s.Unlock()
	s.logger.Tracef(format, args...)
}

// Debug logs a debug message thread-safely
func (s *SafeLogger) Debug(args ...interface{}) {
	s.Lock()
	defer s.Unlock()
	s.logger.Debug(args...)
}

// Debugf logs a formatted debug message thread-safely
func (s *SafeLogger) Debugf(format string, args ...interface{}) {
	s.Lock()
	defer s.Unlock()
	s.logger.Debugf(format, args...)
}

// Info logs an info message thread-safely
func (s *SafeLogger) Info(args ...interface{}) {
	s.Lock()
	defer s.Unlock()
	s.logger.Info(args...)
}

// Infof logs a formatted info message thread-safely
func (s *SafeLogger) Infof(format string, args ...interface{}) {
	s.Lock()
	defer s.Unlock()
	s.logger.Infof(format, args...)
}

// Warn logs a warning message thread-safely
func (s *SafeLogger) Warn(args ...interface{}) {
	s.Lock()
	defer s.Unlock()
	s.logger.Warn(args...)
}

// Warnf logs a formatted warning message thread-safely
func (s *SafeLogger) Warnf(format string, args ...interface{}) {
	s.Lock()
	defer s.Unlock()
	s.logger.Warnf(format, args...)
}

// Error logs an error message thread-safely
func (s *SafeLogger) Error(args ...interface{}) {
	s.Lock()
	defer s.Unlock()
	s.logger.Error(args...)
}

// Errorf logs a formatted error message thread-safely
func (s *SafeLogger) Errorf(format string, args ...interface{}) {
	s.Lock()
	defer s.Unlock()
	s.logger.Errorf(format, args...)
}

// WithField adds a field to the logger thread-safely
func (s *SafeLogger) WithField(key string, value interface{}) logrus.Ext1FieldLogger {
	s.RLock()
	defer s.RUnlock()
	return s.logger.WithField(key, value)
}

// WithFields adds multiple fields to the logger thread-safely
func (s *SafeLogger) WithFields(fields logrus.Fields) logrus.Ext1FieldLogger {
	s.RLock()
	defer s.RUnlock()
	return s.logger.WithFields(fields)
}

// UpdateLogger updates the logger with new fields thread-safely
func (s *SafeLogger) UpdateLogger(fields logrus.Fields) error {
	s.Lock()
	defer s.Unlock()

	newFields := make(logrus.Fields)
	for k, v := range fields {
		if reflect.TypeOf(v).Kind() == reflect.Map && reflect.TypeOf(v).Key().Kind() == reflect.String {
			newMap := make(map[string]interface{})
			err := copier.Copy(&newMap, v)
			if err != nil {
				return err
			}
			newFields[k] = newMap
		} else {
			newFields[k] = v
		}
	}

	s.logger = s.logger.WithFields(newFields)

	return nil
}

// UpdateLoggerWithWarnOnFailure updates the logger with new fields thread-safely and logs a warning if it fails.
func (s *SafeLogger) UpdateLoggerWithWarnOnFailure(fields logrus.Fields, msg string) {
	if err := s.UpdateLogger(fields); err != nil {
		s.WithError(err).Warn(msg)
	}
}
</file>

<file path="pkg/spdk/disk/aio/aio.go">
package aio

import (
	"fmt"
	"io"
	"os"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"

	commontypes "github.com/longhorn/go-common-libs/types"
	spdkclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
	spdktypes "github.com/longhorn/go-spdk-helper/pkg/spdk/types"
	spdkutil "github.com/longhorn/go-spdk-helper/pkg/util"

	"github.com/longhorn/go-spdk-helper/pkg/jsonrpc"

	"github.com/longhorn/longhorn-spdk-engine/pkg/spdk/disk"
)

type DiskDriverAio struct {
}

func init() {
	driver := &DiskDriverAio{}
	disk.RegisterDiskDriver(string(commontypes.DiskDriverAio), driver)
}

func (d *DiskDriverAio) DiskCreate(spdkClient *spdkclient.Client, diskName, diskPath string, blockSize uint64) (string, error) {
	if err := validateDiskCreation(spdkClient, diskPath); err != nil {
		return "", errors.Wrap(err, "failed to validate disk creation")
	}

	return spdkClient.BdevAioCreate(diskPath, diskName, blockSize)
}

func (d *DiskDriverAio) DiskDelete(spdkClient *spdkclient.Client, diskName, diskPath string) (deleted bool, err error) {
	if _, err = spdkClient.BdevAioDelete(diskName); err != nil {
		if !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return false, err
		}
	}
	return true, nil
}

func (d *DiskDriverAio) DiskGet(spdkClient *spdkclient.Client, diskName, diskPath string, timeout uint64) ([]spdktypes.BdevInfo, error) {
	return spdkClient.BdevAioGet(diskName, timeout)
}

func validateDiskCreation(spdkClient *spdkclient.Client, diskPath string) error {
	ok, err := spdkutil.IsBlockDevice(diskPath)
	if err != nil {
		return errors.Wrap(err, "failed to check if disk is a block device")
	}
	if !ok {
		return errors.Wrapf(err, "disk %v is not a block device", diskPath)
	}

	size, err := getDiskDeviceSize(diskPath)
	if err != nil {
		return errors.Wrap(err, "failed to get disk size")
	}
	if size == 0 {
		return fmt.Errorf("disk %v size is 0", diskPath)
	}

	executor, err := spdkutil.NewExecutor(commontypes.ProcDirectory)
	if err != nil {
		return errors.Wrapf(err, "failed to get the executor for AIO disk %v", diskPath)
	}
	if spdkutil.IsBlockDeviceInUse(diskPath, executor) {
		return fmt.Errorf("disk %v is in use (filesystem or partition table is detected). Wipe all data on the disk and repeat create request", diskPath)
	}

	return nil
}

func getDiskDeviceSize(path string) (int64, error) {
	file, err := os.Open(path)
	if err != nil {
		return 0, errors.Wrapf(err, "failed to open %s", path)
	}
	defer func() {
		if errClose := file.Close(); errClose != nil {
			logrus.WithError(errClose).Errorf("Failed to close disk device %s", path)
		}
	}()

	pos, err := file.Seek(0, io.SeekEnd)
	if err != nil {
		return 0, errors.Wrapf(err, "failed to seek %s", path)
	}
	return pos, nil
}
</file>

<file path="pkg/spdk/disk/nvme/nvme.go">
package nvme

import (
	"strings"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"

	commontypes "github.com/longhorn/go-common-libs/types"
	spdkclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
	spdksetup "github.com/longhorn/go-spdk-helper/pkg/spdk/setup"
	spdktypes "github.com/longhorn/go-spdk-helper/pkg/spdk/types"
	helperutil "github.com/longhorn/go-spdk-helper/pkg/util"

	"github.com/longhorn/longhorn-spdk-engine/pkg/spdk/disk"
)

const (
	// Timeouts for disk bdev
	diskCtrlrLossTimeoutSec  = 30
	diskReconnectDelaySec    = 2
	diskFastIOFailTimeoutSec = 15
	diskTransportAckTimeout  = 10
	diskKeepAliveTimeoutMs   = 10000
	diskMultipath            = "disable"
)

type DiskDriverNvme struct {
}

func init() {
	driver := &DiskDriverNvme{}
	disk.RegisterDiskDriver(string(commontypes.DiskDriverNvme), driver)
}

func (d *DiskDriverNvme) DiskCreate(spdkClient *spdkclient.Client, diskName, diskPath string, blockSize uint64) (string, error) {
	// TODO: validate the diskPath
	executor, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	if err != nil {
		return "", errors.Wrapf(err, "failed to get the executor for NVMe disk create %v", diskPath)
	}

	_, err = spdksetup.Bind(diskPath, "", executor)
	if err != nil {
		return "", errors.Wrapf(err, "failed to bind NVMe disk %v", diskPath)
	}
	defer func() {
		if err != nil {
			logrus.WithError(err).Warnf("Unbinding NVMe disk %v since failed to attach", diskPath)

			_, errUnbind := spdksetup.Unbind(diskPath, executor)
			if errUnbind != nil {
				logrus.WithError(errUnbind).Warnf("Failed to unbind NVMe disk %v since failed to attach", diskPath)
			}
		}
	}()
	bdevs, err := spdkClient.BdevNvmeAttachController(diskName, "", diskPath, "", "PCIe", "",
		diskCtrlrLossTimeoutSec, diskReconnectDelaySec, diskFastIOFailTimeoutSec, diskMultipath)
	if err != nil {
		return "", errors.Wrapf(err, "failed to attach NVMe disk %v", diskPath)
	}
	if len(bdevs) == 0 {
		return "", errors.Errorf("failed to attach NVMe disk %v", diskPath)
	}
	return bdevs[0], nil
}

func (d *DiskDriverNvme) DiskDelete(spdkClient *spdkclient.Client, diskName, diskPath string) (deleted bool, err error) {
	executor, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	if err != nil {
		return false, errors.Wrapf(err, "failed to get the executor for NVMe disk %v deletion", diskName)
	}

	controllers, err := spdkClient.BdevNvmeGetControllers("")
	if err != nil {
		return false, errors.Wrap(err, "failed to get NVMe controllers")
	}

	for _, controller := range controllers {
		for _, ctrl := range controller.Ctrlrs {
			if ctrl.Trid.Traddr == diskPath && strings.ToLower(string(ctrl.Trid.Trtype)) == "pcie" {
				logrus.Infof("Detaching NVMe controller %v", controller.Name)
				_, err = spdkClient.BdevNvmeDetachController(controller.Name)
				if err != nil {
					logrus.WithError(err).Warnf("Failed to detach NVMe controller %v", controller.Name)
				}
				break
			}
		}
	}

	_, err = spdksetup.Unbind(diskPath, executor)
	if err != nil {
		logrus.WithError(err).Warnf("Failed to unbind NVMe disk %v", diskPath)
	}
	return true, nil
}

func (d *DiskDriverNvme) DiskGet(spdkClient *spdkclient.Client, diskName, diskPath string, timeout uint64) ([]spdktypes.BdevInfo, error) {
	bdevs, err := spdkClient.BdevGetBdevs("", 0)
	if err != nil {
		return nil, errors.Wrap(err, "failed to get bdevs")
	}
	foundBdevs := []spdktypes.BdevInfo{}
	for _, bdev := range bdevs {
		if bdev.DriverSpecific == nil {
			continue
		}
		if bdev.DriverSpecific.Nvme == nil {
			continue
		}
		nvmes := *bdev.DriverSpecific.Nvme
		for _, nvme := range nvmes {
			if nvme.PciAddress == diskPath {
				foundBdevs = append(foundBdevs, bdev)
			}
		}
	}
	return foundBdevs, nil
}
</file>

<file path="pkg/spdk/disk/virtio-blk/virtio-blk.go">
package virtioblk

import (
	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"

	"github.com/longhorn/go-spdk-helper/pkg/jsonrpc"

	commontypes "github.com/longhorn/go-common-libs/types"
	spdkclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
	spdksetup "github.com/longhorn/go-spdk-helper/pkg/spdk/setup"
	spdktypes "github.com/longhorn/go-spdk-helper/pkg/spdk/types"
	helperutil "github.com/longhorn/go-spdk-helper/pkg/util"

	"github.com/longhorn/longhorn-spdk-engine/pkg/spdk/disk"
)

type DiskDriverVirtioBlk struct {
}

func init() {
	driver := &DiskDriverVirtioBlk{}
	disk.RegisterDiskDriver(string(commontypes.DiskDriverVirtioBlk), driver)
}

func (d *DiskDriverVirtioBlk) DiskCreate(spdkClient *spdkclient.Client, diskName, diskPath string, blockSize uint64) (string, error) {
	// TODO: validate the diskPath
	executor, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	if err != nil {
		return "", errors.Wrapf(err, "failed to get the executor for virtio-blk disk create %v", diskPath)
	}

	_, err = spdksetup.Bind(diskPath, "", executor)
	if err != nil {
		return "", errors.Wrapf(err, "failed to bind virtio-blk disk %v", diskPath)
	}
	defer func() {
		if err != nil {
			logrus.WithError(err).Warnf("Unbinding virtio-blk disk %v since failed to attach", diskPath)

			_, errUnbind := spdksetup.Unbind(diskPath, executor)
			if errUnbind != nil {
				logrus.WithError(errUnbind).Warnf("Failed to unbind virtio-blk disk %v since failed to attach", diskPath)
			}
		}
	}()

	bdevs, err := spdkClient.BdevVirtioAttachController(diskName, "pci", diskPath, "blk")
	if err != nil {
		return "", errors.Wrapf(err, "failed to attach virtio-blk disk %v", diskPath)
	}
	if len(bdevs) == 0 {
		return "", errors.Errorf("failed to attach virtio-blk disk %v", diskPath)
	}
	return bdevs[0], nil
}

func (d *DiskDriverVirtioBlk) DiskDelete(spdkClient *spdkclient.Client, diskName, diskPath string) (deleted bool, err error) {
	executor, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	if err != nil {
		return false, errors.Wrapf(err, "failed to get the executor for virtio-blk disk %v deletion", diskName)
	}

	_, err = spdkClient.BdevVirtioDetachController(diskName)
	if err != nil {
		if !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return false, errors.Wrapf(err, "failed to detach virtio-blk disk %v", diskName)
		}
	}

	_, err = spdksetup.Unbind(diskPath, executor)
	if err != nil {
		return false, errors.Wrapf(err, "failed to unbind virtio-blk disk %v", diskPath)
	}
	return true, nil
}

func (d *DiskDriverVirtioBlk) DiskGet(spdkClient *spdkclient.Client, diskName, diskPath string, timeout uint64) ([]spdktypes.BdevInfo, error) {
	return spdkClient.BdevGetBdevs(diskName, timeout)
}
</file>

<file path="pkg/spdk/disk/virtio-scsi/virtio-scsi.go">
package virtioscsi

import (
	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"

	"github.com/longhorn/go-spdk-helper/pkg/jsonrpc"

	commontypes "github.com/longhorn/go-common-libs/types"
	spdkclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
	spdksetup "github.com/longhorn/go-spdk-helper/pkg/spdk/setup"
	spdktypes "github.com/longhorn/go-spdk-helper/pkg/spdk/types"
	helperutil "github.com/longhorn/go-spdk-helper/pkg/util"

	"github.com/longhorn/longhorn-spdk-engine/pkg/spdk/disk"
)

type DiskDriverVirtioScsi struct {
}

func init() {
	driver := &DiskDriverVirtioScsi{}
	disk.RegisterDiskDriver(string(commontypes.DiskDriverVirtioScsi), driver)
}

func (d *DiskDriverVirtioScsi) DiskCreate(spdkClient *spdkclient.Client, diskName, diskPath string, blockSize uint64) (string, error) {
	// TODO: validate the diskPath
	executor, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	if err != nil {
		return "", errors.Wrapf(err, "failed to get the executor for virtio-scsi disk create %v", diskPath)
	}

	_, err = spdksetup.Bind(diskPath, "", executor)
	if err != nil {
		return "", errors.Wrapf(err, "failed to bind virtio-scsi disk %v", diskPath)
	}
	defer func() {
		if err != nil {
			logrus.WithError(err).Warnf("Unbinding virtio-scsi disk %v since failed to attach", diskPath)

			_, errUnbind := spdksetup.Unbind(diskPath, executor)
			if errUnbind != nil {
				logrus.WithError(errUnbind).Warnf("Failed to unbind virtio-scsi disk %v since failed to attach", diskPath)
			}
		}
	}()

	bdevs, err := spdkClient.BdevVirtioAttachController(diskName, "pci", diskPath, "scsi")
	if err != nil {
		return "", errors.Wrapf(err, "failed to attach virtio-scsi disk %v", diskPath)
	}
	if len(bdevs) == 0 {
		return "", errors.Errorf("failed to attach virtio-scsi disk %v", diskPath)
	}
	return bdevs[0], nil
}

func (d *DiskDriverVirtioScsi) DiskDelete(spdkClient *spdkclient.Client, diskName, diskPath string) (deleted bool, err error) {
	executor, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	if err != nil {
		return false, errors.Wrapf(err, "failed to get the executor for virtio-scsi disk %v deletion", diskName)
	}

	_, err = spdkClient.BdevVirtioDetachController(diskName)
	if err != nil {
		if !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return false, errors.Wrapf(err, "failed to detach virtio-scsi disk %v", diskName)
		}
	}

	_, err = spdksetup.Unbind(diskPath, executor)
	if err != nil {
		return false, errors.Wrapf(err, "failed to unbind virtio-scsi disk %v", diskPath)
	}
	return true, nil
}

func (d *DiskDriverVirtioScsi) DiskGet(spdkClient *spdkclient.Client, diskName, diskPath string, timeout uint64) ([]spdktypes.BdevInfo, error) {
	return spdkClient.BdevGetBdevs(diskName, timeout)
}
</file>

<file path="pkg/spdk/disk/driver.go">
package disk

import (
	"fmt"

	"github.com/cockroachdb/errors"

	spdkclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
	spdktypes "github.com/longhorn/go-spdk-helper/pkg/spdk/types"
)

type DiskDriver interface {
	DiskCreate(*spdkclient.Client, string, string, uint64) (string, error)
	DiskDelete(*spdkclient.Client, string, string) (bool, error)
	DiskGet(*spdkclient.Client, string, string, uint64) ([]spdktypes.BdevInfo, error)
}

var (
	diskDrivers map[string]DiskDriver
)

func init() {
	diskDrivers = make(map[string]DiskDriver)
}

func RegisterDiskDriver(diskDriver string, ops DiskDriver) {
	diskDrivers[diskDriver] = ops
}

func DiskCreate(spdkClient *spdkclient.Client, diskName, diskPath, diskDriver string, blockSize uint64) (string, error) {
	driver, ok := diskDrivers[diskDriver]
	if !ok {
		return "", fmt.Errorf("disk driver %s is not registered", diskDriver)
	}

	return driver.DiskCreate(spdkClient, diskName, diskPath, blockSize)
}

func DiskDelete(spdkClient *spdkclient.Client, diskName, diskPath, diskDriver string) (bool, error) {
	driver, ok := diskDrivers[diskDriver]
	if !ok {
		return false, fmt.Errorf("disk driver %s is not registered", diskDriver)
	}

	return driver.DiskDelete(spdkClient, diskName, diskPath)
}

func DiskGet(spdkClient *spdkclient.Client, diskName, diskPath, diskDriver string, timeout uint64) ([]spdktypes.BdevInfo, error) {
	if diskDriver == "" {
		if !isBDF(diskPath) {
			return spdkClient.BdevGetBdevs(diskName, 0)
		}
		bdevs, err := spdkClient.BdevGetBdevs("", 0)
		if err != nil {
			return nil, errors.Wrapf(err, "failed to get bdevs")
		}
		foundBdevs := []spdktypes.BdevInfo{}
		for _, bdev := range bdevs {
			if bdev.DriverSpecific == nil {
				continue
			}
			if bdev.DriverSpecific.Nvme == nil {
				if diskName == bdev.Name {
					foundBdevs = append(foundBdevs, bdev)
				}
			} else {
				nvmes := *bdev.DriverSpecific.Nvme
				for _, nvme := range nvmes {
					if nvme.PciAddress == diskPath {
						foundBdevs = append(foundBdevs, bdev)
					}
				}
			}
		}
		return foundBdevs, nil
	}

	driver, ok := diskDrivers[diskDriver]
	if !ok {
		return nil, fmt.Errorf("disk driver %s is not registered", diskDriver)
	}

	return driver.DiskGet(spdkClient, diskName, diskPath, timeout)
}
</file>

<file path="pkg/spdk/disk/types_test.go">
package disk

import (
	"fmt"
	"testing"

	. "gopkg.in/check.v1"
)

func Test(t *testing.T) { TestingT(t) }

type TestSuite struct{}

var _ = Suite(&TestSuite{})

func (s *TestSuite) TestIsVfioPci(c *C) {
	fmt.Println("Testing isVfioPci function with various driver strings")

	testCases := []struct {
		name     string
		driver   string
		expected bool
	}{
		{
			name:     "VfioPci with hyphen",
			driver:   "vfio-pci",
			expected: true,
		},
		{
			name:     "VfioPci with underscore",
			driver:   "vfio_pci",
			expected: true,
		},
		{
			name:     "Non-VfioPci driver",
			driver:   "virtio-pci",
			expected: false,
		},
		{
			name:     "Empty driver",
			driver:   "",
			expected: false,
		},
	}
	for _, tc := range testCases {
		c.Logf("Running test case: %s", tc.name)
		result := isVfioPci(tc.driver)
		c.Assert(result, Equals, tc.expected, Commentf("Expected %v for driver %s, got %v", tc.expected, tc.driver, result))
	}
}

func (s *TestSuite) TestIsUioPciGeneric(c *C) {
	fmt.Println("Testing isUioPciGeneric function with various driver strings")

	testCases := []struct {
		name     string
		driver   string
		expected bool
	}{
		{
			name:     "UioPciGeneric with hyphen",
			driver:   "uio-pci-generic",
			expected: true,
		},
		{
			name:     "UioPciGeneric with underscore",
			driver:   "uio_pci_generic",
			expected: true,
		},
		{
			name:     "Non-UioPciGeneric driver",
			driver:   "virtio-pci",
			expected: false,
		},
		{
			name:     "Empty driver",
			driver:   "",
			expected: false,
		},
	}
	for _, tc := range testCases {
		c.Logf("Running test case: %s", tc.name)
		result := isUioPciGeneric(tc.driver)
		c.Assert(result, Equals, tc.expected, Commentf("Expected %v for driver %s, got %v", tc.expected, tc.driver, result))
	}
}
</file>

<file path="pkg/spdk/disk/types.go">
package disk

import (
	"fmt"
	"regexp"
	"slices"
	"strings"

	"github.com/cockroachdb/errors"

	commontypes "github.com/longhorn/go-common-libs/types"
	spdksetup "github.com/longhorn/go-spdk-helper/pkg/spdk/setup"
	helpertypes "github.com/longhorn/go-spdk-helper/pkg/types"
	helperutil "github.com/longhorn/go-spdk-helper/pkg/util"

	"github.com/longhorn/longhorn-spdk-engine/pkg/util"
)

type BlockDiskSubsystem string

const (
	BlockDiskSubsystemVirtio = BlockDiskSubsystem("virtio")
	BlockDiskSubsystemPci    = BlockDiskSubsystem("pci")
	BlockDiskSubsystemNvme   = BlockDiskSubsystem("nvme")
	BlockDiskSubsystemScsi   = BlockDiskSubsystem("scsi")
)

type BlockDiskType string

const (
	BlockDiskTypeDisk = BlockDiskType("disk")
	BlockDiskTypeLoop = BlockDiskType("loop")
)

func GetDiskDriver(diskDriver commontypes.DiskDriver, diskPathOrBdf string) (commontypes.DiskDriver, error) {
	if isBDF(diskPathOrBdf) {
		return getDiskDriverForBDF(diskDriver, diskPathOrBdf)
	}

	return getDiskDriverForPath(diskDriver, diskPathOrBdf)
}

// isVfioPci checks if the given driver is vfio_pci or a variant of it.
func isVfioPci(driver string) bool {
	normalized := strings.ReplaceAll(driver, "-", "_")
	return normalized == string(commontypes.DiskDriverVfioPci)
}

// isUioPciGeneric checks if the given driver is uio_pci_generic or a variant of it.
func isUioPciGeneric(driver string) bool {
	normalized := strings.ReplaceAll(driver, "-", "_")
	return normalized == string(commontypes.DiskDriverUioPciGeneric)
}

func getDiskDriverForBDF(diskDriver commontypes.DiskDriver, bdf string) (commontypes.DiskDriver, error) {
	executor, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	if err != nil {
		return "", errors.Wrapf(err, "failed to get the executor for disk driver detection")
	}

	diskStatus, err := spdksetup.GetDiskStatus(bdf, executor)
	if err != nil {
		return "", errors.Wrapf(err, "failed to get disk status for BDF %s", bdf)
	}

	switch diskDriver {
	case commontypes.DiskDriverAuto:
		diskPath := ""
		if !isVfioPci(diskStatus.Driver) && !isUioPciGeneric(diskStatus.Driver) {
			devName, err := util.GetDevNameFromBDF(bdf)
			if err != nil {
				return "", errors.Wrapf(err, "failed to get device name from BDF %s", bdf)
			}
			diskPath = fmt.Sprintf("/dev/%s", devName)
		}
		return getDriverForAuto(diskStatus, diskPath)
	case commontypes.DiskDriverAio, commontypes.DiskDriverNvme, commontypes.DiskDriverVirtioScsi, commontypes.DiskDriverVirtioBlk:
		return diskDriver, nil
	default:
		return commontypes.DiskDriverNone, fmt.Errorf("unsupported disk driver %s for BDF %s", diskDriver, bdf)
	}
}

func getDriverForAuto(diskStatus *helpertypes.DiskStatus, diskPath string) (commontypes.DiskDriver, error) {
	// SPDK supports various types of disks, including NVMe, virtio-blk, and virtio-scsi.
	//
	// NVMe disks can be managed by either NVMe bdev or AIO bdev.
	// VirtIO disks can be managed by virtio-blk, virtio-scsi, or AIO bdev.
	//
	// To use the correct bdev,  need to identify the disk type.
	// Here's how to identify the disk type:
	// - If a block device uses the subsystems virtio and pci, it's a virtio-blk disk.
	// - If it uses the subsystems virtio, pci, and scsi, it's a virtio-scsi disk.
	switch diskStatus.Driver {
	case string(commontypes.DiskDriverNvme):
		return commontypes.DiskDriverNvme, nil
	case string(commontypes.DiskDriverVirtioPci):
		blockdevice, err := util.GetBlockDevice(diskPath)
		if err != nil {
			return commontypes.DiskDriverNone, errors.Wrapf(err, "failed to get blockdevice info for %s", diskPath)
		}

		if slices.Contains(blockdevice.Subsystems, string(BlockDiskSubsystemVirtio)) && slices.Contains(blockdevice.Subsystems, string(BlockDiskSubsystemPci)) {
			diskDriver := commontypes.DiskDriverVirtioBlk
			if slices.Contains(blockdevice.Subsystems, string(BlockDiskSubsystemScsi)) {
				diskDriver = commontypes.DiskDriverVirtioScsi
			}
			return diskDriver, nil
		}

		return commontypes.DiskDriverNone, fmt.Errorf("unsupported disk driver %s for disk path %s", diskStatus.Driver, diskPath)
	default:
		return commontypes.DiskDriverNone, fmt.Errorf("unsupported disk driver %s for disk path %s", diskStatus.Driver, diskPath)
	}
}

func getDiskDriverForPath(diskDriver commontypes.DiskDriver, diskPath string) (commontypes.DiskDriver, error) {
	switch diskDriver {
	case commontypes.DiskDriverAuto, commontypes.DiskDriverAio:
		return commontypes.DiskDriverAio, nil
	default:
		return commontypes.DiskDriverNone, fmt.Errorf("unsupported disk driver %s for disk path %s", diskDriver, diskPath)
	}
}

func isBDF(addr string) bool {
	bdfFormat := "[a-f0-9]{4}:[a-f0-9]{2}:[a-f0-9]{2}\\.[a-f0-9]{1}"
	bdfPattern := regexp.MustCompile(bdfFormat)
	return bdfPattern.MatchString(addr)
}
</file>

<file path="pkg/spdk/backing_image.go">
package spdk

import (
	"context"
	"fmt"
	"net"
	"net/url"
	"os"
	"strconv"
	"strings"
	"sync"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/go-spdk-helper/pkg/initiator"
	"github.com/longhorn/go-spdk-helper/pkg/jsonrpc"
	"github.com/longhorn/types/pkg/generated/spdkrpc"

	commonbitmap "github.com/longhorn/go-common-libs/bitmap"
	commonnet "github.com/longhorn/go-common-libs/net"
	commontypes "github.com/longhorn/go-common-libs/types"
	spdkclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
	spdktypes "github.com/longhorn/go-spdk-helper/pkg/spdk/types"
	helpertypes "github.com/longhorn/go-spdk-helper/pkg/types"
	helperutil "github.com/longhorn/go-spdk-helper/pkg/util"

	"github.com/longhorn/longhorn-spdk-engine/pkg/types"
	"github.com/longhorn/longhorn-spdk-engine/pkg/util"

	safelog "github.com/longhorn/longhorn-spdk-engine/pkg/log"
)

type BackingImage struct {
	sync.RWMutex

	ctx context.Context

	// Name of the BackingImage, e.g. "parrot"
	Name             string
	BackingImageUUID string
	LvsName          string
	LvsUUID          string
	Size             uint64
	ProcessedSize    int64
	ExpectedChecksum string

	IsExposed bool
	Port      int32

	// We need to create a snapshot from the BackingImage lvol so we can create the replica from that snapshot.
	Snapshot *Lvol
	// This is the lvol alias for the snapshot lvol, e.g. "n1v2disk/bi-parrot-disk-${lvsuuid}"
	Alias string

	Progress        int32
	State           types.BackingImageState
	CurrentChecksum string
	ErrorMsg        string

	UpdateCh chan interface{}

	log *safelog.SafeLogger
}

func ServiceBackingImageToProtoBackingImage(bi *BackingImage) *spdkrpc.BackingImage {
	res := &spdkrpc.BackingImage{
		Name:             bi.Name,
		BackingImageUuid: bi.BackingImageUUID,
		LvsName:          bi.LvsName,
		LvsUuid:          bi.LvsUUID,
		Size:             bi.Size,
		ExpectedChecksum: bi.ExpectedChecksum,
		Snapshot:         nil,
		Progress:         bi.Progress,
		State:            string(bi.State),
		CurrentChecksum:  bi.CurrentChecksum,
		ErrorMsg:         bi.ErrorMsg,
	}
	if bi.Snapshot != nil {
		res.Snapshot = ServiceBackingImageLvolToProtoBackingImageLvol(bi.Snapshot)
	}
	return res
}

func NewBackingImage(ctx context.Context, backingImageName, backingImageUUID, lvsUUID string, size uint64, checksum string, updateCh chan interface{}) *BackingImage {
	log := logrus.StandardLogger().WithFields(logrus.Fields{
		"backingImagename": backingImageName,
		"lvsUUID":          lvsUUID,
	})

	return &BackingImage{
		ctx:              ctx,
		Name:             backingImageName,
		BackingImageUUID: backingImageUUID,
		LvsUUID:          lvsUUID,
		Size:             size,
		ExpectedChecksum: checksum,
		State:            types.BackingImageStateStarting,
		UpdateCh:         updateCh,
		log:              safelog.NewSafeLogger(log),
	}
}

// Create initiates the backing image, prepare the lvol, copy the data from the local backing file and create the snapshot.
func (bi *BackingImage) Create(spdkClient *spdkclient.Client, superiorPortAllocator *commonbitmap.Bitmap, fromAddress string, srcLvsUUID string) (ret *spdkrpc.BackingImage, err error) {
	updateRequired := true

	bi.Lock()
	defer func() {
		bi.Unlock()

		if updateRequired {
			bi.UpdateCh <- nil
		}
	}()
	if bi.State == types.BackingImageStateReady {
		updateRequired = false
		return nil, grpcstatus.Errorf(grpccodes.AlreadyExists, "backing image %v already exists and running", bi.Name)
	}
	if bi.State != types.BackingImageStateStarting {
		updateRequired = false
		return nil, grpcstatus.Errorf(grpccodes.Internal, "invalid state %s for backing image %s creation", bi.State, bi.Name)
	}

	go func() {
		err := bi.prepareBackingImageSnapshot(spdkClient, superiorPortAllocator, fromAddress, srcLvsUUID)
		if err != nil {
			bi.log.WithError(err).Error("Failed to create backing image")
		}
		// update the backing image after preparing
		bi.UpdateCh <- nil
	}()

	return ServiceBackingImageToProtoBackingImage(bi), nil
}

func (bi *BackingImage) Get() (pBackingImage *spdkrpc.BackingImage) {
	bi.RLock()
	defer bi.RUnlock()
	return ServiceBackingImageToProtoBackingImage(bi)
}

func (bi *BackingImage) Delete(spdkClient *spdkclient.Client, superiorPortAllocator *commonbitmap.Bitmap) (err error) {
	updateRequired := false

	bi.Lock()
	defer func() {
		if err != nil {
			bi.log.WithError(err).Errorf("Failed to delete backing image")
			bi.State = types.BackingImageStateFailed
			bi.ErrorMsg = err.Error()
			updateRequired = true
		}
		bi.Unlock()

		if updateRequired {
			bi.UpdateCh <- nil
		}
	}()

	// blindly unexpose the lvol bdevs
	backingImageSnapLvolName := GetBackingImageSnapLvolName(bi.Name, bi.LvsUUID)
	if err = spdkClient.StopExposeBdev(helpertypes.GetNQN(backingImageSnapLvolName)); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return errors.Wrapf(err, "failed to unexpose lvol bdev %v when deleting backing image %v", backingImageSnapLvolName, bi.Name)
	}

	backingImageTempHeadName := GetBackingImageTempHeadLvolName(bi.Name, bi.LvsUUID)
	if err = spdkClient.StopExposeBdev(helpertypes.GetNQN(backingImageTempHeadName)); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return errors.Wrapf(err, "failed to unexpose lvol bdev %v when deleting backing image %v", backingImageTempHeadName, bi.Name)
	}
	bi.IsExposed = false
	if err := superiorPortAllocator.ReleaseRange(bi.Port, bi.Port); err != nil {
		return errors.Wrapf(err, "failed to release port %v after when deleting backing image %v", bi.Port, bi.Name)
	}
	bi.Port = 0

	biTempHeadAlias := fmt.Sprintf("%s/%s", bi.LvsName, GetBackingImageTempHeadLvolName(bi.Name, bi.LvsUUID))
	if _, err := spdkClient.BdevLvolDelete(biTempHeadAlias); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return err
	}

	if _, err := spdkClient.BdevLvolDelete(bi.Alias); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return err
	}

	bi.log.Info("Deleted backing image")
	updateRequired = true
	return nil
}

func (bi *BackingImage) ValidateAndUpdate(spdkClient *spdkclient.Client) (err error) {
	updateRequired := false
	bi.Lock()
	defer func() {
		bi.Unlock()

		if updateRequired {
			bi.UpdateCh <- nil
		}
	}()

	// Backing image normal state transition: starting -> in-progress -> ready/failed
	// If backing image is in ready or failed state, that means we are validating a record already cached.
	// We check if the corresponding lvol existence.
	// If backing image is in starting and in-progress state, we don't need to do anything because it is still preparing.
	// For uncache backing image lvol, the state will be pending, and we will need to reconstruct the record.
	if bi.State == types.BackingImageStateReady || bi.State == types.BackingImageStateFailed {
		defer func() {
			if err != nil {
				bi.State = types.BackingImageStateFailed
				bi.ErrorMsg = err.Error()
				updateRequired = true
			}
		}()

		// check if the lvol sill exists
		bdevLvolList, err := spdkClient.BdevLvolGet(bi.Alias, 0)
		if err != nil {
			return err
		}
		if len(bdevLvolList) != 1 {
			return fmt.Errorf("zero or multiple snap lvols with alias %v found when validating", bi.Alias)
		}
		return nil
	}

	if bi.State == types.BackingImageStateInProgress || bi.State == types.BackingImageStateStarting {
		return nil
	}

	biLvolName := GetBackingImageSnapLvolName(bi.Name, bi.LvsUUID)

	lvsName, err := GetLvsNameByUUID(spdkClient, bi.LvsUUID)
	if err != nil {
		return errors.Wrapf(err, "failed to get the lvs name for backing image %v with lvs uuid %v", bi.Name, bi.LvsUUID)
	}

	bi.log.UpdateLoggerWithWarnOnFailure(logrus.Fields{
		"backingImagename": bi.Name,
		"lvsName":          bi.LvsName,
		"lvsUUID":          bi.LvsUUID,
	}, "Failed to update logger")

	bi.LvsName = lvsName

	bi.Alias = spdktypes.GetLvolAlias(bi.LvsName, biLvolName)
	bdevLvolList, err := spdkClient.BdevLvolGet(bi.Alias, 0)
	if err != nil {
		return err
	}
	if len(bdevLvolList) != 1 {
		return fmt.Errorf("zero or multiple snap lvols with alias %v found after lvol snapshot", bi.Alias)
	}
	snapSvcLvol := BdevLvolInfoToServiceLvol(&bdevLvolList[0])
	bi.Snapshot = snapSvcLvol
	state, err := GetSnapXattr(spdkClient, bi.Alias, types.LonghornBackingImageSnapshotAttrPrepareState)
	if err != nil {
		return errors.Wrapf(err, "failed to get the prepare state for backing image snapshot %v", bi.Name)
	}
	bi.State = types.BackingImageState(state)

	// TODO: recheck the checksum when pick up the backing image snapshot lvol
	bi.CurrentChecksum = bi.ExpectedChecksum
	bi.Progress = 100
	updateRequired = true

	return nil
}

func (bi *BackingImage) BackingImageExpose(spdkClient *spdkclient.Client, superiorPortAllocator *commonbitmap.Bitmap) (exposedSnapshotLvolAddress string, err error) {
	bi.log.Infof("Exposing backing image %v for syncing", bi.Name)

	updateRequired := false

	bi.Lock()
	defer func() {
		bi.Unlock()
		if updateRequired {
			bi.UpdateCh <- nil
		}
	}()

	defer func() {
		if err != nil {
			bi.log.WithError(err).Warnf("Failed to expose the backing image snapshot lvol")

			// We don't need to handle exposed bdev here since it is the last step.
			// If it failed to expose the bdev, we only need to release the port.
			opErr := superiorPortAllocator.ReleaseRange(bi.Port, bi.Port)
			if opErr != nil {
				err = errors.Wrapf(err, "Failed to release port %v with error %v", bi.Port, opErr)
			} else {
				bi.Port = 0
			}
			updateRequired = true
		}
	}()

	if bi.State != types.BackingImageStateReady {
		return "", fmt.Errorf("invalid state %v for backing image %s to be exposed for syncing", bi.State, bi.Name)
	}
	backingImageSnapLvolName := GetBackingImageSnapLvolName(bi.Name, bi.LvsUUID)

	snapLvol := bi.Snapshot
	if snapLvol == nil {
		return "", fmt.Errorf("cannot find snapshot for the backing image %s to be exposed for syncing", bi.Name)
	}

	// Expose the bdev using nvmf
	podIP, err := commonnet.GetIPForPod()
	if err != nil {
		return "", err
	}

	if bi.IsExposed {
		exposedSnapshotLvolAddress = net.JoinHostPort(podIP, strconv.Itoa(int(bi.Port)))
		bi.log.Infof("Backing image lvol bdev %v is already exposed with exposedSnapshotLvolAddress %v", backingImageSnapLvolName, exposedSnapshotLvolAddress)
		return exposedSnapshotLvolAddress, nil
	}

	port, _, err := superiorPortAllocator.AllocateRange(int32(types.BackingImagePortCount))
	if err != nil {
		return "", err
	}
	bi.Port = port
	bi.log.Infof("Allocated port %v", port)

	executor, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	if err != nil {
		return "", errors.Wrapf(err, "failed to create executor")
	}

	subsystemNQN, controllerName, err := exposeSnapshotLvolBdev(spdkClient, bi.LvsName, backingImageSnapLvolName, podIP, port, executor)
	if err != nil {
		bi.log.WithError(err).Errorf("Failed to expose lvol bdev")
		return "", err
	}
	bi.IsExposed = true

	exposedSnapshotLvolAddress = net.JoinHostPort(podIP, strconv.Itoa(int(port)))
	bi.log.Infof("Exposed lvol bdev %v, subsystemNQN %v, controllerName %v, exposedSnapshotLvolAddress: %v", backingImageSnapLvolName, subsystemNQN, controllerName, exposedSnapshotLvolAddress)

	updateRequired = true

	return exposedSnapshotLvolAddress, nil
}

func (bi *BackingImage) BackingImageUnexpose(spdkClient *spdkclient.Client, superiorPortAllocator *commonbitmap.Bitmap) (err error) {
	bi.log.Infof("Stop exposing backing image %v", bi.Name)

	updateRequired := false

	bi.Lock()
	defer func() {
		bi.Unlock()

		if updateRequired {
			bi.UpdateCh <- nil
		}
	}()

	if !bi.IsExposed {
		return nil
	}

	backingImageSnapLvolName := GetBackingImageSnapLvolName(bi.Name, bi.LvsUUID)
	// Unexpose the bdev
	bi.log.Info("Unexposing lvol bdev")
	if err := spdkClient.StopExposeBdev(helpertypes.GetNQN(backingImageSnapLvolName)); err != nil {
		return errors.Wrapf(err, "Failed to unexpose lvol bdev %v", backingImageSnapLvolName)
	}
	bi.IsExposed = false

	if err := superiorPortAllocator.ReleaseRange(bi.Port, bi.Port); err != nil {
		return errors.Wrapf(err, "Failed to release port %v after failed to create backing image", bi.Port)
	}
	bi.Port = 0
	updateRequired = true

	return
}

func checkIsSourceFromBIM(fromAddress string) bool {
	return strings.HasPrefix(fromAddress, "http")
}

func (bi *BackingImage) UpdateProgress(processedSize int64) {
	bi.updateProgress(processedSize)
	bi.UpdateCh <- nil
}

func (bi *BackingImage) updateProgress(processedSize int64) {
	bi.Lock()
	defer bi.Unlock()

	if bi.State == types.BackingImageStateStarting {
		bi.State = types.BackingImageStateInProgress
	}
	if bi.State == types.BackingImageStateReady {
		return
	}

	bi.ProcessedSize = bi.ProcessedSize + processedSize
	if bi.Size > 0 {
		bi.Progress = int32((float32(bi.ProcessedSize) / float32(bi.Size)) * 100)
	}
}

func (bi *BackingImage) prepareBackingImageSnapshot(spdkClient *spdkclient.Client, superiorPortAllocator *commonbitmap.Bitmap, fromAddress string, srcLvsUUID string) (err error) {
	// If we already create the temp head lvol, no matter it fails to prepare or not, we should create the snapshot for it.
	// The state will be recorded in the xattr of the snapshot lvol so when the node is rebooted, we can pick it up and reconstruct the record.
	// Controller will handle the failed backing image copy by deleting it and recreating it.
	biTempHeadUUID := ""

	defer func() {
		bi.Lock()
		defer bi.Unlock()

		if bi.IsExposed {
			bi.log.Info("Unexposing lvol bdev")
			backingImageTempHeadName := GetBackingImageTempHeadLvolName(bi.Name, bi.LvsUUID)
			if err = spdkClient.StopExposeBdev(helpertypes.GetNQN(backingImageTempHeadName)); err != nil {
				bi.log.WithError(err).Errorf("Failed to unexpose lvol bdev %v after failed to create backing image", backingImageTempHeadName)
			}
			bi.IsExposed = false
		}
		if bi.Port != 0 {
			if err = superiorPortAllocator.ReleaseRange(bi.Port, bi.Port); err != nil {
				bi.log.WithError(err).Errorf("Failed to release port %v after failed to create backing image", bi.Port)
			}
			bi.Port = 0
		}

		if err == nil {
			if bi.State != types.BackingImageStateInProgress {
				err = fmt.Errorf("invalid state %v for backing image %s creation after processing", bi.State, bi.Name)
			}

			if bi.Size > 0 && bi.ProcessedSize != int64(bi.Size) {
				err = fmt.Errorf("processed data size %v does not match the expected file size %v", bi.ProcessedSize, bi.Size)
			}

			if bi.CurrentChecksum != bi.ExpectedChecksum {
				err = fmt.Errorf("current checksum %v does not match the expected checksum %v", bi.CurrentChecksum, bi.ExpectedChecksum)
			}
		}

		if err != nil {
			bi.log.WithError(err).Errorf("Failed to create backing image %v", bi.Name)
			if bi.State != types.BackingImageStateFailed {
				bi.State = types.BackingImageStateFailed
			}
			bi.ErrorMsg = err.Error()
		} else {
			bi.Progress = 100
			bi.State = types.BackingImageStateReady
			bi.ErrorMsg = ""
		}

		if biTempHeadUUID != "" {
			// Create a snapshot with backing image name from the backing image temp head lvol
			// the snapshot lvol name will be "bi-${biName}-disk-${lvsUUID}"
			// and the alias will be "${lvsName}/bi-${biName}-disk-${lvsUUID}"

			createSnapshotErr := bi.createSnapshotFromTempHead(spdkClient, biTempHeadUUID)
			if createSnapshotErr != nil {
				bi.State = types.BackingImageStateFailed
				bi.Progress = 0
				if bi.ErrorMsg != "" {
					bi.ErrorMsg = fmt.Sprintf("%v; %v", bi.ErrorMsg, createSnapshotErr.Error())
				} else {
					bi.ErrorMsg = createSnapshotErr.Error()
				}
			}

			backingImageTempHeadName := GetBackingImageTempHeadLvolName(bi.Name, bi.LvsUUID)
			if _, opErr := spdkClient.BdevLvolDelete(biTempHeadUUID); opErr != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(opErr) {
				bi.log.WithError(opErr).Errorf("Failed to delete the temp head %v of backing image ", backingImageTempHeadName)
			}
		}
	}()

	bi.Lock()
	lvsName, err := GetLvsNameByUUID(spdkClient, bi.LvsUUID)
	if err != nil {
		return errors.Wrapf(err, "failed to get the lvs name for backing image %v with lvs uuid %v", bi.Name, bi.LvsUUID)
	}
	bi.LvsName = lvsName

	bi.log.UpdateLoggerWithWarnOnFailure(logrus.Fields{
		"backingImagename": bi.Name,
		"lvsName":          bi.LvsName,
		"lvsUUID":          bi.LvsUUID,
	}, "Failed to update logger")

	bi.Unlock()

	// backingImageTempHeadName will be "bi-${biName}-disk-${lvsUUID}-temp-head"
	backingImageTempHeadName := GetBackingImageTempHeadLvolName(bi.Name, bi.LvsUUID)
	biTempHeadUUID, err = spdkClient.BdevLvolCreate("", bi.LvsUUID, backingImageTempHeadName, util.BytesToMiB(bi.Size), "", true)
	if err != nil {
		return err
	}
	bdevLvolList, err := spdkClient.BdevLvolGet(biTempHeadUUID, 0)
	if err != nil {
		return err
	}
	if len(bdevLvolList) < 1 {
		return fmt.Errorf("cannot find lvol %v after creation", backingImageTempHeadName)
	}
	bi.log.Infof("Created a head lvol %v for the new backing image", backingImageTempHeadName)

	podIP, err := commonnet.GetIPForPod()
	if err != nil {
		return err
	}
	port, _, err := superiorPortAllocator.AllocateRange(int32(types.BackingImagePortCount))
	if err != nil {
		return err
	}
	bi.Lock()
	bi.Port = port
	bi.Unlock()
	bi.log.Infof("Allocated port %v", port)

	executor, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	if err != nil {
		return errors.Wrapf(err, "failed to create executor")
	}
	subsystemNQN, controllerName, err := exposeSnapshotLvolBdev(spdkClient, bi.LvsName, backingImageTempHeadName, podIP, port, executor)
	if err != nil {
		bi.log.WithError(err).Errorf("Failed to expose head lvol")
		return err
	}
	bi.Lock()
	bi.IsExposed = true
	bi.Unlock()
	bi.log.Infof("Exposed head lvol %v, subsystemNQN %v, controllerName %v", backingImageTempHeadName, subsystemNQN, controllerName)

	nvmeTCPInfo := &initiator.NVMeTCPInfo{
		SubsystemNQN: helpertypes.GetNQN(backingImageTempHeadName),
	}
	headInitiator, err := initiator.NewInitiator(backingImageTempHeadName, initiator.HostProc, nvmeTCPInfo, nil)
	if err != nil {
		return errors.Wrapf(err, "failed to create NVMe initiator for head lvol %v", backingImageTempHeadName)
	}
	if _, err := headInitiator.StartNvmeTCPInitiator(podIP, strconv.Itoa(int(port)), true, true); err != nil {
		return errors.Wrapf(err, "failed to start NVMe initiator for head lvol %v", backingImageTempHeadName)
	}
	bi.log.Infof("Created NVMe initiator for head lvol %v", backingImageTempHeadName)

	headFh, err := os.OpenFile(headInitiator.Endpoint, os.O_RDWR, 0666)
	defer func() {
		// Stop the initiator
		if errClose := headFh.Close(); errClose != nil {
			bi.log.WithError(errClose).Error("Failed to close the backing image")
		}
		bi.log.Info("Stopping NVMe initiator")
		if _, opErr := headInitiator.Stop(nil, true, true, false); opErr != nil {
			bi.log.WithError(opErr).Error("Failed to stop the backing image head NVMe initiator")
		}
	}()
	if err != nil {
		return errors.Wrapf(err, "failed to open NVMe device %v for lvol bdev %v", headInitiator.Endpoint, backingImageTempHeadName)
	}

	// An SPDK backing image should only be created by downloading it from BIM if it's a first copy,
	// or by syncing it from another SPDK server.
	isSourceFromBIM := checkIsSourceFromBIM(fromAddress)
	if isSourceFromBIM {
		if err := bi.prepareFromURL(headFh, fromAddress); err != nil {
			bi.log.WithError(err).Warnf("Failed to prepare the backing image %v from URL %v", bi.Name, fromAddress)
			return errors.Wrapf(err, "failed to prepare the backing image %v from URL %v", bi.Name, fromAddress)
		}
	} else {
		if err := bi.prepareFromSync(headFh, fromAddress, srcLvsUUID); err != nil {
			bi.log.WithError(err).Warnf("Failed to prepare the backing image %v by syncing from %v and srcLvsUUID %v", bi.Name, fromAddress, srcLvsUUID)
			return errors.Wrapf(err, "failed to prepare the backing image %v by syncing from  %v and srcLvsUUID %v", bi.Name, fromAddress, srcLvsUUID)
		}
	}

	currentChecksum, err := util.GetFileChecksum(headInitiator.Endpoint)
	if err != nil {
		return errors.Wrapf(err, "failed to get the current checksum of the backing image %v target device %v", bi.Name, headInitiator.Endpoint)
	}
	bi.log.Infof("Get the current checksum of the backing image %v: %v", bi.Name, currentChecksum)
	bi.Lock()
	bi.CurrentChecksum = currentChecksum
	bi.Unlock()

	return nil
}

func (bi *BackingImage) createSnapshotFromTempHead(spdkClient *spdkclient.Client, biTempHeadUUID string) (err error) {
	ne, err := helperutil.NewExecutor(commontypes.HostProcDirectory)
	if err != nil {
		bi.log.WithError(err).Warnf("Failed to get the executor for snapshot backing image %v, will skip the sync and continue", bi.Name)
	} else {
		bi.log.Infof("Requesting system sync before snapshot backin image %v", bi.Name)
		// TODO: only sync the device path rather than all filesystems
		if _, err := ne.Execute(nil, "sync", []string{}, SyncTimeout); err != nil {
			// sync should never fail though, so it more like due to the nsenter
			bi.log.WithError(err).Errorf("WARNING: failed to sync for snapshot backing image %v, will skip the sync and continue", bi.Name)
		}
	}

	biSnapLvolName := GetBackingImageSnapLvolName(bi.Name, bi.LvsUUID)
	bi.Alias = spdktypes.GetLvolAlias(bi.LvsName, biSnapLvolName)

	var xattrs []spdkclient.Xattr
	checksum := spdkclient.Xattr{
		Name:  types.LonghornBackingImageSnapshotAttrChecksum,
		Value: bi.ExpectedChecksum,
	}
	xattrs = append(xattrs, checksum)
	backingImageUUID := spdkclient.Xattr{
		Name:  types.LonghornBackingImageSnapshotAttrUUID,
		Value: bi.BackingImageUUID,
	}
	xattrs = append(xattrs, backingImageUUID)
	prepareState := spdkclient.Xattr{
		Name:  types.LonghornBackingImageSnapshotAttrPrepareState,
		Value: string(bi.State),
	}
	xattrs = append(xattrs, prepareState)

	snapUUID, err := spdkClient.BdevLvolSnapshot(biTempHeadUUID, biSnapLvolName, xattrs)
	if err != nil {
		return err
	}

	bdevLvolList, err := spdkClient.BdevLvolGet(snapUUID, 0)
	if err != nil {
		return err
	}

	if len(bdevLvolList) != 1 {
		return fmt.Errorf("zero or multiple snap lvols with UUID %s found after lvol snapshot", snapUUID)
	}

	snapSvcLvol := BdevLvolInfoToServiceLvol(&bdevLvolList[0])
	bi.Snapshot = snapSvcLvol

	return nil
}

func (bi *BackingImage) prepareFromURL(targetFh *os.File, fromAddress string) (err error) {
	httpHandler := util.HTTPHandler{}

	// Parse the base URL into a URL object
	parsedURL, err := url.Parse(fromAddress)
	if err != nil {
		bi.log.WithError(err).Error("Failed to parse the URL")
		return errors.Wrapf(err, "failed to parse the URL %v", fromAddress)
	}
	// Add query parameters
	query := parsedURL.Query()
	query.Set("forV2Creation", "true")
	parsedURL.RawQuery = query.Encode()

	size, err := httpHandler.GetSizeFromURL(parsedURL.String())
	if err != nil {
		return errors.Wrapf(err, "failed to get the file size from %v", fromAddress)
	}
	if size != int64(bi.Size) {
		return errors.Wrapf(err, "download file %v size %v is not the same as the backing image size %v", fromAddress, size, bi.Size)
	}

	if _, err := httpHandler.DownloadFromURL(bi.ctx, parsedURL.String(), targetFh, bi); err != nil {
		return errors.Wrapf(err, "failed to download the file from %v", fromAddress)
	}
	return nil
}

func (bi *BackingImage) prepareFromSync(targetFh *os.File, fromAddress, srcLvsUUID string) (err error) {
	if fromAddress == "" || srcLvsUUID == "" {
		return errors.Wrapf(err, "missing required source backing image service address %v or source lvsUUID %v", fromAddress, srcLvsUUID)
	}
	srcBackingImageServiceCli, err := GetServiceClient(fromAddress)
	if err != nil {
		return errors.Wrapf(err, "failed to init the source backing image spdk service client")
	}
	defer func() {
		if errClose := srcBackingImageServiceCli.Close(); errClose != nil {
			bi.log.WithError(errClose).Error("Failed to close the source backing image spdk service client")
		}
	}()
	exposedSnapshotLvolAddress, err := srcBackingImageServiceCli.BackingImageExpose(bi.Name, srcLvsUUID)
	if err != nil {
		return errors.Wrapf(err, "failed to expose the source backing image %v", bi.Name)
	}
	externalSnapshotLvolName := GetBackingImageSnapLvolName(bi.Name, srcLvsUUID)
	defer func() {
		// Unexpose the source snapshot
		if err := srcBackingImageServiceCli.BackingImageUnexpose(bi.Name, srcLvsUUID); err != nil {
			bi.log.WithError(err).Warnf("failed to unsexpose the source backing image %v", bi.Name)
		}
	}()

	srcIP, srcPort, err := splitHostPort(exposedSnapshotLvolAddress)
	if err != nil {
		return errors.Wrapf(err, "failed to split host and port from address %v", exposedSnapshotLvolAddress)
	}
	_, _, err = discoverAndConnectNVMeTarget(srcIP, srcPort, maxRetries, retryInterval)
	if err != nil {
		return errors.Wrapf(err, "failed to connect to NVMe target for source backing image %v in lvsUUID %v with address %v", bi.Name, srcLvsUUID, exposedSnapshotLvolAddress)
	}

	bi.log.Info("Creating NVMe initiator for source backing image %v", bi.Name)
	nvmeTCPInfo := &initiator.NVMeTCPInfo{
		SubsystemNQN: helpertypes.GetNQN(externalSnapshotLvolName),
	}
	i, err := initiator.NewInitiator(externalSnapshotLvolName, initiator.HostProc, nvmeTCPInfo, nil)
	if err != nil {
		return errors.Wrapf(err, "failed to create NVMe initiator for source backing image %v in lvsUUID %v with address %v", bi.Name, srcLvsUUID, exposedSnapshotLvolAddress)
	}
	if _, err := i.StartNvmeTCPInitiator(srcIP, strconv.Itoa(int(srcPort)), true, true); err != nil {
		return errors.Wrapf(err, "failed to start NVMe initiator for source backing image %v in lvsUUID %v with address %v", bi.Name, srcLvsUUID, exposedSnapshotLvolAddress)
	}

	bi.log.Infof("Opening NVMe device %v", i.Endpoint)
	srcFh, err := os.OpenFile(i.Endpoint, os.O_RDWR, 0666)
	defer func() {
		if errClose := srcFh.Close(); errClose != nil {
			bi.log.WithError(errClose).Error("Failed to close the source backing image")
		}
		// Stop the source initiator
		bi.log.Info("Stopping NVMe initiator")
		if _, err := i.Stop(nil, true, true, false); err != nil {
			bi.log.WithError(err).Warnf("failed to stop NVMe initiator")
		}
	}()
	if err != nil {
		return errors.Wrapf(err, "failed to open NVMe device %v for source backing image %v in lvsUUID %v with address %v", i.Endpoint, bi.Name, srcLvsUUID, exposedSnapshotLvolAddress)
	}

	ctx, cancel := context.WithCancel(bi.ctx)
	defer cancel()
	_, err = util.IdleTimeoutCopy(ctx, cancel, srcFh, targetFh, bi, false)
	if err != nil {
		return errors.Wrapf(err, "failed to copy the source backing image %v in lvsUUID %v with address %v", bi.Name, srcLvsUUID, exposedSnapshotLvolAddress)
	}

	return nil
}

func cleanupOrphanBackingImageTempHead(spdkClient *spdkclient.Client, lvsName, backingImageTempHeadName string) error {
	if err := spdkClient.StopExposeBdev(helpertypes.GetNQN(backingImageTempHeadName)); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return errors.Wrapf(err, "Failed to unexpose orphan backing image temp head %v", backingImageTempHeadName)
	}

	biTempHeadAlias := fmt.Sprintf("%s/%s", lvsName, backingImageTempHeadName)
	if _, err := spdkClient.BdevLvolDelete(biTempHeadAlias); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return errors.Wrapf(err, "Failed to delete orphan backing image temp head %v", backingImageTempHeadName)
	}
	return nil
}
</file>

<file path="pkg/spdk/backup_restore_test.go">
package spdk

import (
	"fmt"

	lhtypes "github.com/longhorn/longhorn-spdk-engine/pkg/types"

	. "gopkg.in/check.v1"
)

// Tests for ensureReplicaModeForInfoUpdate, which is called by
// checkAndUpdateInfoFromReplicaNoLock — a function now invoked at the end of
// BackupRestoreFinish to refresh engine state from replica info.
func (s *TestSuite) TestEnsureReplicaModeForInfoUpdateRWQualifies(c *C) {
	fmt.Println("Testing ensureReplicaModeForInfoUpdate: RW mode qualifies for info update")

	e := NewEngine("engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))
	rs := &EngineReplicaStatus{Mode: lhtypes.ModeRW, Address: "10.0.0.1:1234"}

	ok := e.ensureReplicaModeForInfoUpdate("replica-1", rs)

	c.Assert(ok, Equals, true)
	c.Assert(rs.Mode, Equals, lhtypes.Mode(lhtypes.ModeRW))
}

func (s *TestSuite) TestEnsureReplicaModeForInfoUpdateWOQualifies(c *C) {
	fmt.Println("Testing ensureReplicaModeForInfoUpdate: WO mode qualifies for info update")

	e := NewEngine("engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))
	rs := &EngineReplicaStatus{Mode: lhtypes.ModeWO, Address: "10.0.0.1:1234"}

	ok := e.ensureReplicaModeForInfoUpdate("replica-1", rs)

	c.Assert(ok, Equals, true)
	c.Assert(rs.Mode, Equals, lhtypes.Mode(lhtypes.ModeWO))
}

func (s *TestSuite) TestEnsureReplicaModeForInfoUpdateERRDoesNotQualify(c *C) {
	fmt.Println("Testing ensureReplicaModeForInfoUpdate: ERR mode does not qualify")

	e := NewEngine("engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))
	rs := &EngineReplicaStatus{Mode: lhtypes.ModeERR, Address: "10.0.0.1:1234"}

	ok := e.ensureReplicaModeForInfoUpdate("replica-1", rs)

	c.Assert(ok, Equals, false)
	c.Assert(rs.Mode, Equals, lhtypes.Mode(lhtypes.ModeERR))
}

func (s *TestSuite) TestEnsureReplicaModeForInfoUpdateUnexpectedModeDowngradesToERR(c *C) {
	fmt.Println("Testing ensureReplicaModeForInfoUpdate: unexpected mode is downgraded to ERR")

	e := NewEngine("engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))
	rs := &EngineReplicaStatus{Mode: lhtypes.Mode("UNKNOWN"), Address: "10.0.0.1:1234"}

	ok := e.ensureReplicaModeForInfoUpdate("replica-1", rs)

	c.Assert(ok, Equals, false)
	c.Assert(rs.Mode, Equals, lhtypes.Mode(lhtypes.ModeERR))
}

// Tests for checkAndUpdateInfoFromReplicaNoLock with edge-case replica maps.
// This function is now called by BackupRestoreFinish after setting replicas
// to ModeRW.
func (s *TestSuite) TestCheckAndUpdateInfoFromReplicaNoLockEmptyMap(c *C) {
	fmt.Println("Testing checkAndUpdateInfoFromReplicaNoLock: empty ReplicaStatusMap does not panic")

	e := NewEngine("engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))
	e.ReplicaStatusMap = map[string]*EngineReplicaStatus{}

	// Should not panic with empty map
	e.checkAndUpdateInfoFromReplicaNoLock()
}

func (s *TestSuite) TestCheckAndUpdateInfoFromReplicaNoLockAllERRSkipped(c *C) {
	fmt.Println("Testing checkAndUpdateInfoFromReplicaNoLock: all-ERR replicas are skipped without network calls")

	e := NewEngine("engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))
	e.ReplicaStatusMap = map[string]*EngineReplicaStatus{
		"replica-1": {Mode: lhtypes.ModeERR, Address: "10.0.0.1:1234"},
		"replica-2": {Mode: lhtypes.ModeERR, Address: "10.0.0.2:1234"},
	}

	// All replicas are ERR, so ensureReplicaModeForInfoUpdate returns false
	// for each one. No inspectReplicaForInfoUpdate or network calls occur.
	e.checkAndUpdateInfoFromReplicaNoLock()

	// Modes remain ERR (not downgraded further)
	c.Assert(e.ReplicaStatusMap["replica-1"].Mode, Equals, lhtypes.Mode(lhtypes.ModeERR))
	c.Assert(e.ReplicaStatusMap["replica-2"].Mode, Equals, lhtypes.Mode(lhtypes.ModeERR))
}
</file>

<file path="pkg/spdk/backup.go">
package spdk

import (
	"encoding/base64"
	"fmt"
	"os"
	"strconv"
	"sync"

	"github.com/0xPolygon/polygon-edge/consensus/polybft/bitmap"
	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"

	"github.com/longhorn/backupstore"
	"github.com/longhorn/go-spdk-helper/pkg/initiator"

	btypes "github.com/longhorn/backupstore/types"
	commonbitmap "github.com/longhorn/go-common-libs/bitmap"
	commonnet "github.com/longhorn/go-common-libs/net"
	commonns "github.com/longhorn/go-common-libs/ns"
	commontypes "github.com/longhorn/go-common-libs/types"
	spdkclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
	helpertypes "github.com/longhorn/go-spdk-helper/pkg/types"
	helperutil "github.com/longhorn/go-spdk-helper/pkg/util"

	"github.com/longhorn/longhorn-spdk-engine/pkg/util"
)

type Fragmap struct {
	Map         bitmap.Bitmap
	ClusterSize uint64
	NumClusters uint64
}

type Backup struct {
	sync.Mutex

	spdkClient *spdkclient.Client

	Name          string
	VolumeName    string
	SnapshotName  string
	replica       *Replica
	fragmap       *Fragmap
	IP            string
	Port          int32
	IsIncremental bool

	BackupURL string
	State     btypes.ProgressState
	Progress  int
	Error     string

	subsystemNQN   string
	controllerName string
	initiator      *initiator.Initiator
	devFh          *os.File
	executor       *commonns.Executor

	log logrus.FieldLogger
}

var _ backupstore.DeltaBlockBackupOperations = (*Backup)(nil)

// NewBackup creates a new backup instance
func NewBackup(spdkClient *spdkclient.Client, backupName, volumeName, snapshotName string, replica *Replica, superiorPortAllocator *commonbitmap.Bitmap) (*Backup, error) {
	log := logrus.WithFields(logrus.Fields{
		"backupName":   backupName,
		"volumeName":   volumeName,
		"snapshotName": snapshotName,
	})

	log.Info("Initializing backup")

	podIP, err := commonnet.GetIPForPod()
	if err != nil {
		return nil, err
	}

	port, _, err := superiorPortAllocator.AllocateRange(1)
	if err != nil {
		return nil, err
	}

	executor, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	if err != nil {
		return nil, errors.Wrapf(err, "failed to create executor")
	}

	return &Backup{
		spdkClient:   spdkClient,
		Name:         backupName,
		VolumeName:   volumeName,
		SnapshotName: snapshotName,
		replica:      replica,
		IP:           podIP,
		Port:         port,
		State:        btypes.ProgressStateInProgress,
		log:          log,
		executor:     executor,
	}, nil
}

// BackupCreate creates the backup
func (b *Backup) BackupCreate(config *backupstore.DeltaBackupConfig) error {
	b.log.Info("Creating backup")

	isIncremental, err := backupstore.CreateDeltaBlockBackup(b.Name, config)
	if err != nil {
		return err
	}

	b.IsIncremental = isIncremental
	return nil
}

// HasSnapshot checks if the snapshot exists
func (b *Backup) HasSnapshot(snapshotName, volumeName string) bool {
	b.log.Info("Checking if snapshot exists")

	b.Lock()
	defer b.Unlock()

	if b.VolumeName != volumeName {
		b.log.Warnf("Invalid state volume [%s] are open, not [%s]", b.VolumeName, volumeName)
		return false
	}

	return b.findIndex(GetReplicaSnapshotLvolName(b.replica.Name, snapshotName)) >= 0
}

// OpenSnapshot opens the snapshot lvol for backup
func (b *Backup) OpenSnapshot(snapshotName, volumeName string) error {
	b.Lock()
	defer b.Unlock()

	b.log.Info("Preparing snapshot lvol bdev for backup")
	frgmap, err := b.newFragmap()
	if err != nil {
		return err
	}
	b.fragmap = frgmap

	lvolName := GetReplicaSnapshotLvolName(b.replica.Name, snapshotName)

	b.replica.Lock()
	defer b.replica.Unlock()

	b.log.Infof("Exposing snapshot lvol bdev %v", lvolName)
	subsystemNQN, controllerName, err := exposeSnapshotLvolBdev(b.spdkClient, b.replica.LvsName, lvolName, b.IP, b.Port, b.executor)
	if err != nil {
		b.log.WithError(err).Errorf("Failed to expose snapshot lvol bdev %v", lvolName)
		return errors.Wrapf(err, "failed to expose snapshot lvol bdev %v", lvolName)
	}
	b.subsystemNQN = subsystemNQN
	b.controllerName = controllerName

	b.log.Infof("Creating NVMe initiator for snapshot lvol bdev %v", lvolName)
	nvmeTCPInfo := &initiator.NVMeTCPInfo{
		SubsystemNQN: helpertypes.GetNQN(lvolName),
	}
	i, err := initiator.NewInitiator(lvolName, initiator.HostProc, nvmeTCPInfo, nil)
	if err != nil {
		return errors.Wrapf(err, "failed to create NVMe initiator for snapshot lvol bdev %v", lvolName)
	}
	if _, err := i.StartNvmeTCPInitiator(b.IP, strconv.Itoa(int(b.Port)), false, true); err != nil {
		return errors.Wrapf(err, "failed to start NVMe initiator for snapshot lvol bdev %v", lvolName)
	}
	b.initiator = i

	b.log.Infof("Opening NVMe device %v", b.initiator.Endpoint)
	devFh, err := os.OpenFile(b.initiator.Endpoint, os.O_RDONLY, 0666)
	if err != nil {
		return errors.Wrapf(err, "failed to open NVMe device %v for snapshot lvol bdev %v", b.initiator.Endpoint, lvolName)
	}
	b.devFh = devFh

	return nil
}

// CompareSnapshot compares the data between two snapshots and returns the mappings
func (b *Backup) CompareSnapshot(snapshotName, compareSnapshotName, volumeName string, blockSize int64) (*btypes.Mappings, error) {
	b.log.Infof("Comparing snapshots from %v to %v", snapshotName, compareSnapshotName)

	lvolName := GetReplicaSnapshotLvolName(b.replica.Name, snapshotName)

	compareLvolName := ""
	if compareSnapshotName != "" {
		compareLvolName = GetReplicaSnapshotLvolName(b.replica.Name, compareSnapshotName)
	}

	b.replica.Lock()
	defer b.replica.Unlock()

	from, to, err := b.findSnapshotRange(lvolName, compareLvolName)
	if err != nil {
		return nil, errors.Wrapf(err, "failed to find snapshot lvol range %v (%v) and %v (%v)",
			lvolName, from, compareLvolName, to)
	}

	// Overlay the fragments of snapshots and store the result in the b.fragmap.Map
	b.log.Infof("Constructing fragment map for snapshot lvols from %v (%v) to %v (%v)", lvolName, from, compareLvolName, to)
	if err := b.constructFragmap(from, to); err != nil {
		return nil, errors.Wrapf(err, "failed to construct fragment map for snapshot lvols from %v (%v) to %v (%v)",
			lvolName, from, compareLvolName, to)
	}

	return b.constructMappings(blockSize), nil
}

// ReadSnapshot reads the data from the block device exposed by NVMe/TCP TCP
func (b *Backup) ReadSnapshot(snapshotName, volumeName string, offset int64, data []byte) error {
	b.Lock()
	defer b.Unlock()

	_, err := b.devFh.ReadAt(data, offset)

	return err
}

func (b *Backup) CloseSnapshot(snapshotName, volumeName string) error {
	b.Lock()
	defer b.Unlock()

	b.log.Infof("Closing NVMe device %v", b.initiator.Endpoint)
	if err := b.devFh.Close(); err != nil {
		return errors.Wrapf(err, "failed to close NVMe device %v", b.initiator.Endpoint)
	}

	b.log.Info("Stopping NVMe initiator")
	if _, err := b.initiator.Stop(nil, true, true, true); err != nil {
		return errors.Wrapf(err, "failed to stop NVMe initiator")
	}

	b.log.Info("Unexposing snapshot lvol bdev")
	lvolName := GetReplicaSnapshotLvolName(b.replica.Name, snapshotName)
	err := b.spdkClient.StopExposeBdev(helpertypes.GetNQN(lvolName))
	if err != nil {
		return errors.Wrapf(err, "failed to unexpose snapshot lvol bdev %v", lvolName)
	}

	return nil
}

// UpdateBackupStatus updates the backup status. The state is first-respected, but if
// - The errString is not empty, the state will be set to error.
// - The progress is 100, the state will be set to complete.
func (b *Backup) UpdateBackupStatus(snapshotName, volumeName string, state string, progress int, url string, errString string) error {
	b.Lock()
	defer b.Unlock()

	b.State = btypes.ProgressState(state)
	b.Progress = progress
	b.BackupURL = url
	b.Error = errString

	if b.Progress == 100 {
		b.State = btypes.ProgressStateComplete
	} else {
		if b.Error != "" {
			b.State = btypes.ProgressStateError
		}
	}

	return nil
}

func (b *Backup) newFragmap() (*Fragmap, error) {
	lvsList, err := b.spdkClient.BdevLvolGetLvstore(b.replica.LvsName, "")
	if err != nil {
		return nil, err
	}
	if len(lvsList) == 0 {
		return nil, errors.Errorf("cannot find lvs %v for volume %v backup creation", b.replica.LvsName, b.VolumeName)
	}
	lvs := lvsList[0]

	if lvs.ClusterSize == 0 || lvs.BlockSize == 0 {
		return nil, errors.Errorf("invalid cluster size %v block size %v lvs %v", lvs.ClusterSize, lvs.BlockSize, b.replica.LvsName)
	}

	if (b.replica.SpecSize % lvs.ClusterSize) != 0 {
		return nil, errors.Errorf("replica size %v is not multiple of cluster size %v", b.replica.SpecSize, lvs.ClusterSize)
	}

	numClusters := b.replica.SpecSize / lvs.ClusterSize

	return &Fragmap{
		ClusterSize: lvs.ClusterSize,
		NumClusters: numClusters,
		// Calculate the number of bytes in the fragmap required considering 8 bits per byte
		Map: make([]byte, (numClusters+7)/8),
	}, nil
}

func (b *Backup) overlayFragmap(fragmap []byte, offset, size uint64) error {
	b.log.Debugf("Overlaying fragment map for offset %v size %v", offset, size)

	startBytes := int((offset / b.fragmap.ClusterSize) / 8)
	if startBytes+len(fragmap) > len(b.fragmap.Map) {
		return fmt.Errorf("invalid start bytes %v and fragmap length %v", startBytes, len(fragmap))
	}

	for i := 0; i < len(fragmap); i++ {
		b.fragmap.Map[startBytes+i] |= fragmap[i]
	}
	return nil
}

func (b *Backup) overlayFragmaps(lvol *Lvol) error {
	// Cluster size is 1 MiB by default, so each byte in fragmap represents 8 clusters.
	// Process 256 bytes at a time to reduce the number of calls.
	batchSize := 256 * (8 * b.fragmap.ClusterSize)

	// Old snapshots remain smaller after expansion; cap fragmap to the lvol size.
	// E.g. a 1Gi snapshot stays 1Gi after expanding the volume to 2Gi.
	effectiveSize := min(b.replica.SpecSize, lvol.SpecSize)

	offset := uint64(0)
	for {
		if offset >= effectiveSize {
			return nil
		}

		size := util.Min(batchSize, effectiveSize-offset)

		result, err := b.spdkClient.BdevLvolGetFragmap(lvol.UUID, uint64(offset), size)
		if err != nil {
			return err
		}

		fragmap, err := base64.StdEncoding.DecodeString(result.Fragmap)
		if err != nil {
			return err
		}

		err = b.overlayFragmap(fragmap, offset, size)
		if err != nil {
			return err
		}

		offset += size
	}
}

func (b *Backup) constructFragmap(from, to int) error {
	for i := from; i > to; i-- {
		lvol := b.replica.ActiveChain[i]
		if lvol != nil {
			b.log.Infof("Overlaying snapshot lvol bdev %v", lvol.Name)
			err := b.overlayFragmaps(lvol)
			if err != nil {
				return errors.Wrapf(err, "failed to overlay fragment map for snapshot lvol bdev %v", lvol.Name)
			}
		}
	}
	return nil
}

func (b *Backup) findSnapshotRange(lvolName, compareLvolName string) (from, to int, err error) {
	from = b.findIndex(lvolName)
	if from < 0 {
		return 0, 0, fmt.Errorf("failed to find snapshot %s in chain", lvolName)
	}

	to = b.findIndex(compareLvolName)
	if to < 0 {
		return 0, 0, fmt.Errorf("failed to find snapshot %s in chain", compareLvolName)
	}

	if from <= to {
		b.log.Warnf("Last backup snapshot %s is not an ancestor of current snapshot %s; performing full backup instead",
			compareLvolName, lvolName)
		to = 0
	}

	if from > len(b.replica.ActiveChain)-1 {
		return 0, 0, fmt.Errorf("invalid to index %v which is greater than the length of active chain %v",
			to, len(b.replica.ActiveChain)-1)
	}

	return from, to, nil
}

func (b *Backup) constructMappings(blockSize int64) *btypes.Mappings {
	b.log.Info("Constructing mappings")

	mappings := &btypes.Mappings{
		BlockSize: blockSize,
	}

	mapping := btypes.Mapping{
		Offset: -1,
	}

	i := uint64(0)
	for i = 0; i < b.fragmap.NumClusters; i++ {
		if b.fragmap.Map.IsSet(uint64(i)) {
			offset := int64(i) * int64(b.fragmap.ClusterSize)
			offset -= (offset % blockSize)
			if mapping.Offset != offset {
				mapping = btypes.Mapping{
					Offset: offset,
					Size:   blockSize,
				}
				mappings.Mappings = append(mappings.Mappings, mapping)
			}
		}
	}

	b.log.Info("Constructed mappings")

	return mappings
}

func (b *Backup) findIndex(lvolName string) int {
	if lvolName == "" {
		// Note that, 0 can be a backing image if ActiveChanin[0] is not nil.
		// Caller should handle this case
		return 0
	}

	for i, lvol := range b.replica.ActiveChain {
		if i == 0 {
			continue
		}
		if lvol.Name == lvolName {
			return i
		}
	}

	return -1
}
</file>

<file path="pkg/spdk/disk.go">
package spdk

import (
	"fmt"
	"path/filepath"
	"sync"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"google.golang.org/protobuf/types/known/emptypb"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/go-spdk-helper/pkg/jsonrpc"
	"github.com/longhorn/types/pkg/generated/spdkrpc"

	commontypes "github.com/longhorn/go-common-libs/types"
	spdkclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
	spdktypes "github.com/longhorn/go-spdk-helper/pkg/spdk/types"
	spdkutil "github.com/longhorn/go-spdk-helper/pkg/util"

	"github.com/longhorn/longhorn-spdk-engine/pkg/util"

	spdkdisk "github.com/longhorn/longhorn-spdk-engine/pkg/spdk/disk"

	_ "github.com/longhorn/longhorn-spdk-engine/pkg/spdk/disk/aio"
	_ "github.com/longhorn/longhorn-spdk-engine/pkg/spdk/disk/nvme"
	_ "github.com/longhorn/longhorn-spdk-engine/pkg/spdk/disk/virtio-blk"
	_ "github.com/longhorn/longhorn-spdk-engine/pkg/spdk/disk/virtio-scsi"
)

const (
	defaultClusterSize = 1 * 1024 * 1024 // 1MB
	defaultBlockSize   = 4096            // 4KB

	hostPrefix = "/host"
)

type DiskState string

const (
	DiskStateUnknown  = DiskState("")
	DiskStateError    = DiskState("error")
	DiskStateReady    = DiskState("ready")
	DiskStateCreating = DiskState("creating")
)

type Disk struct {
	sync.RWMutex

	Name string
	UUID string

	DiskDriver string
	DiskID     string
	DiskPath   string
	DiskType   string

	TotalSize   int64
	FreeSize    int64
	TotalBlocks int64
	FreeBlocks  int64
	BlockSize   int64
	ClusterSize int64

	State DiskState
}

func (d *Disk) GetState() DiskState {
	d.RLock()
	defer d.RUnlock()
	return d.State
}

func NewDisk(diskName, diskUUID, diskPath, diskDriver string, blockSize int64) *Disk {
	return &Disk{
		DiskPath:  diskPath,
		BlockSize: blockSize,
		DiskType:  DiskTypeBlock,
		State:     DiskStateCreating,
	}
}

func (d *Disk) DiskCreate(spdkClient *spdkclient.Client, diskName, diskUUID, diskPath, diskDriver string, blockSize int64) (err error) {
	log := logrus.WithFields(logrus.Fields{
		"diskName":   diskName,
		"diskUUID":   diskUUID,
		"diskPath":   diskPath,
		"blockSize":  blockSize,
		"diskDriver": diskDriver,
	})

	log.Info("Creating disk")

	d.Lock()
	defer func() {
		if err != nil {
			d.State = DiskStateError
			log.WithError(err).Error("Failed to create disk")
		} else {
			d.State = DiskStateReady
			log.Info("Created disk successfully")
		}
		d.Unlock()
	}()

	if diskName == "" || diskPath == "" {
		return grpcstatus.Error(grpccodes.InvalidArgument, "disk name and disk path are required")
	}

	exactDiskDriver, err := spdkdisk.GetDiskDriver(commontypes.DiskDriver(diskDriver), diskPath)
	if err != nil {
		log.WithError(err).Error("Failed to determine disk driver")
		return grpcstatus.Errorf(grpccodes.InvalidArgument, "failed to get disk driver for disk %q: %v", diskName, err)
	}
	d.DiskDriver = string(exactDiskDriver)

	lvstoreUUID, err := addBlockDevice(spdkClient, diskName, diskUUID, diskPath, exactDiskDriver, blockSize)
	if err != nil {
		log.WithError(err).Error("Failed to add block device")
		return grpcstatus.Errorf(grpccodes.Internal, "failed to add disk block device: %v", err)
	}

	diskID, err := getDiskID(diskPath, exactDiskDriver)
	if err != nil {
		log.WithError(err).Error("Failed to get disk ID")
		return grpcstatus.Errorf(grpccodes.Internal, "failed to get disk ID for %q: %v", diskName, err)
	}
	d.DiskID = diskID

	if err := d.lvstoreToDisk(spdkClient, "", lvstoreUUID); err != nil {
		log.WithError(err).Error("Failed to update disk from lvstore")
		return grpcstatus.Errorf(grpccodes.Internal, "failed to update disk from lvstore: %v", err)
	}

	return nil
}

func (d *Disk) DiskDelete(spdkClient *spdkclient.Client, diskName, diskUUID, diskPath, diskDriver string) (ret *emptypb.Empty, err error) {
	log := logrus.WithFields(logrus.Fields{
		"diskName":   diskName,
		"diskUUID":   diskUUID,
		"diskPath":   diskPath,
		"diskDriver": diskDriver,
	})

	log.Info("Deleting disk")

	d.Lock()
	defer d.Unlock()

	defer func() {
		if err != nil {
			log.WithError(err).Error("Failed to delete disk")
		} else {
			log.Info("Deleted disk")
		}
	}()

	if diskName == "" {
		return &emptypb.Empty{}, grpcstatus.Error(grpccodes.InvalidArgument, "disk name is required")
	}

	if diskUUID != "" {
		lvstores, err := spdkClient.BdevLvolGetLvstore("", diskUUID)
		if err != nil {
			if !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
				return nil, errors.Wrapf(err, "failed to get lvstore with UUID %v", diskUUID)
			}
		}

		if len(lvstores) == 0 {
			log.Infof("Lvstore not found for disk %v (UUID %v); treating as already deleted", diskName, diskUUID)
		} else if lvstores[0].UUID != diskUUID {
			log.Warnf("Lvstore UUID mismatch (expected %v, found %v); proceed with bdev deletion", diskUUID, lvstores[0].UUID)
		}
	} else {
		// The disk is not successfully created in creation stage because the diskUUID is not provided,
		// so we blindly use the diskName as the bdevName here.
		log.Warn("Disk UUID is not provided, blindly delete the disk")
	}

	if diskDriver == "" {
		diskDriver = d.DiskDriver
	}
	if _, err := spdkdisk.DiskDelete(spdkClient, diskName, diskPath, diskDriver); err != nil {
		return nil, errors.Wrapf(err, "failed to delete disk %v", diskName)
	}

	return &emptypb.Empty{}, nil
}

type DeviceInfo struct {
	DeviceDriver string `json:"device_driver"`
}

func (d *Disk) updateDiskFromLvstoreNoLock(spdkClient *spdkclient.Client, diskName, diskPath, diskDriver string) (string, error) {
	bdevs, err := spdkdisk.DiskGet(spdkClient, diskName, diskPath, diskDriver, 0)
	if err != nil {
		if !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return "", grpcstatus.Errorf(grpccodes.Internal, "failed to get bdev with name %q: %v", diskName, err)
		}
	}
	if len(bdevs) == 0 {
		return "", grpcstatus.Errorf(grpccodes.NotFound, "no disk bdev found with name %q", diskName)
	}

	var (
		targetBdev      *spdktypes.BdevInfo
		exactDiskDriver commontypes.DiskDriver
	)

	for i := range bdevs {
		bdev := &bdevs[i]
		switch bdev.ProductName {
		case spdktypes.BdevProductNameAio:
			if bdev.DriverSpecific != nil {
				diskPath = util.RemovePrefix(bdev.DriverSpecific.Aio.FileName, hostPrefix)
				exactDiskDriver = commontypes.DiskDriverAio
				targetBdev = bdev
			}
		case spdktypes.BdevProductNameVirtioBlk:
			exactDiskDriver = commontypes.DiskDriverVirtioBlk
			targetBdev = bdev
		case spdktypes.BdevProductNameVirtioScsi:
			exactDiskDriver = commontypes.DiskDriverVirtioScsi
			targetBdev = bdev
		case spdktypes.BdevProductNameNvme:
			exactDiskDriver = commontypes.DiskDriverNvme
			targetBdev = bdev
		}

		if targetBdev != nil {
			break
		}
	}

	if targetBdev == nil {
		return "", grpcstatus.Errorf(grpccodes.NotFound, "no matching disk bdev found for %q", diskName)
	}

	diskID, err := getDiskID(diskPath, exactDiskDriver)
	if err != nil {
		return "", errors.Wrapf(err, "failed to get disk ID")
	}

	d.DiskID = diskID
	d.DiskDriver = string(exactDiskDriver)

	return targetBdev.Name, nil
}

func (d *Disk) DiskGet(spdkClient *spdkclient.Client, diskName, diskPath, diskDriver string) (ret *spdkrpc.Disk, err error) {
	log := logrus.WithFields(logrus.Fields{
		"diskName":   diskName,
		"diskPath":   diskPath,
		"diskDriver": diskDriver,
	})

	log.Trace("Getting disk info")

	defer func() {
		if err != nil {
			log.WithError(err).Error("Failed to get disk info")
		} else {
			log.Trace("Got disk info")
		}
	}()

	if diskName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "disk name is required")
	}

	d.RLock()
	if d.State == DiskStateCreating || d.State == DiskStateError {
		defer d.RUnlock()
		return &spdkrpc.Disk{
			Id:          d.DiskID,
			Name:        d.Name,
			Uuid:        d.UUID,
			Path:        d.DiskPath,
			Type:        DiskTypeBlock,
			Driver:      d.DiskDriver,
			TotalSize:   d.TotalSize,
			FreeSize:    d.FreeSize,
			TotalBlocks: d.TotalBlocks,
			FreeBlocks:  d.FreeBlocks,
			BlockSize:   d.BlockSize,
			ClusterSize: d.ClusterSize,
			State:       string(d.State),
		}, nil
	}
	d.RUnlock()

	d.Lock()
	diskBdevName, err := d.updateDiskFromLvstoreNoLock(spdkClient, diskName, diskPath, diskDriver)
	if err != nil {
		d.Unlock()
		return nil, err
	}
	if err := d.lvstoreToDisk(spdkClient, diskBdevName, ""); err != nil {
		d.Unlock()
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to update disk from lvstore: %v", err)
	}
	d.Unlock()

	d.RLock()
	defer d.RUnlock()
	return &spdkrpc.Disk{
		Id:          d.DiskID,
		Name:        d.Name,
		Uuid:        d.UUID,
		Path:        d.DiskPath,
		Type:        DiskTypeBlock,
		Driver:      d.DiskDriver,
		TotalSize:   d.TotalSize,
		FreeSize:    d.FreeSize,
		TotalBlocks: d.TotalBlocks,
		FreeBlocks:  d.FreeBlocks,
		BlockSize:   d.BlockSize,
		ClusterSize: d.ClusterSize,
		State:       string(d.State),
	}, nil
}

func getDiskPath(path string) string {
	return filepath.Join(hostPrefix, path)
}

func getDiskIDFromDeviceNumber(filename string) (string, error) {
	executor, err := spdkutil.NewExecutor(commontypes.ProcDirectory)
	if err != nil {
		return "", err
	}

	dev, err := spdkutil.DetectDevice(filename, executor)
	if err != nil {
		return "", errors.Wrapf(err, "failed to detect disk device %v", filename)
	}

	return fmt.Sprintf("%d-%d", dev.Major, dev.Minor), nil
}

func validateAioDiskCreation(spdkClient *spdkclient.Client, diskPath string, diskDriver commontypes.DiskDriver) error {
	diskID, err := getDiskIDFromDeviceNumber(getDiskPath(diskPath))
	if err != nil {
		return errors.Wrap(err, "failed to get disk device number")
	}

	bdevs, err := spdkdisk.DiskGet(spdkClient, "", "", string(diskDriver), 0)
	if err != nil {
		return errors.Wrap(err, "failed to get disk bdevs")
	}

	for _, bdev := range bdevs {
		id, err := getDiskIDFromDeviceNumber(bdev.DriverSpecific.Aio.FileName)
		if err != nil {
			return errors.Wrap(err, "failed to get disk device number")
		}

		if id == diskID {
			return fmt.Errorf("disk %v is already used by disk bdev %v", diskPath, bdev.Name)
		}
	}

	return nil
}

func addBlockDevice(spdkClient *spdkclient.Client, diskName, diskUUID, originalDiskPath string, diskDriver commontypes.DiskDriver, blockSize int64) (string, error) {
	log := logrus.WithFields(logrus.Fields{
		"diskName":   diskName,
		"diskUUID":   diskUUID,
		"diskPath":   originalDiskPath,
		"diskDriver": diskDriver,
		"blockSize":  blockSize,
	})

	diskPath := originalDiskPath
	if diskDriver == commontypes.DiskDriverAio {
		if err := validateAioDiskCreation(spdkClient, diskPath, diskDriver); err != nil {
			return "", errors.Wrap(err, "failed to validate disk creation")
		}
		diskPath = getDiskPath(originalDiskPath)
	}

	log.Info("Creating disk bdev")

	bdevName, err := spdkdisk.DiskCreate(spdkClient, diskName, diskPath, string(diskDriver), uint64(blockSize))
	if err != nil {
		if !jsonrpc.IsJSONRPCRespErrorFileExists(err) {
			return "", errors.Wrapf(err, "failed to create disk bdev")
		}
	}

	bdevs, err := spdkdisk.DiskGet(spdkClient, bdevName, diskPath, "", 0)
	if err != nil {
		return "", errors.Wrapf(err, "failed to get disk bdev")
	}
	if len(bdevs) == 0 {
		return "", fmt.Errorf("cannot find disk bdev with name %v", bdevName)
	}
	if len(bdevs) > 1 {
		return "", fmt.Errorf("found multiple disk bdevs with name %v", bdevName)
	}
	bdev := bdevs[0]

	// Name of the lvstore is the same as the name of the disk bdev
	lvstoreName := bdev.Name

	log.Infof("Creating lvstore %v", lvstoreName)

	lvstores, err := spdkClient.BdevLvolGetLvstore("", "")
	if err != nil {
		return "", errors.Wrapf(err, "failed to get lvstores")
	}

	for _, lvstore := range lvstores {
		if lvstore.BaseBdev != bdevName {
			continue
		}

		if diskUUID != "" && diskUUID != lvstore.UUID {
			continue
		}

		log.Infof("Found an existing lvstore %v", lvstore.Name)
		if lvstore.Name == lvstoreName {
			return lvstore.UUID, nil
		}

		// Rename the existing lvstore to the name of the disk bdev if the UUID matches
		log.Infof("Renaming the existing lvstore %v to %v", lvstore.Name, lvstoreName)
		renamed, err := spdkClient.BdevLvolRenameLvstore(lvstore.Name, lvstoreName)
		if err != nil {
			return "", errors.Wrapf(err, "failed to rename lvstore %v to %v", lvstore.Name, lvstoreName)
		}
		if !renamed {
			return "", fmt.Errorf("failed to rename lvstore %v to %v", lvstore.Name, lvstoreName)
		}
		return lvstore.UUID, nil
	}

	if diskUUID == "" {
		log.Infof("Creating a new lvstore %v", lvstoreName)
		return spdkClient.BdevLvolCreateLvstore(bdev.Name, lvstoreName, defaultClusterSize)
	}

	// The lvstore should be created before, but it cannot be found now.
	return "", grpcstatus.Error(grpccodes.NotFound, fmt.Sprintf("cannot find lvstore with UUID %v", diskUUID))
}

func getDiskID(diskPath string, diskDriver commontypes.DiskDriver) (string, error) {
	var err error

	diskID := diskPath
	if diskDriver == commontypes.DiskDriverAio {
		diskID, err = getDiskIDFromDeviceNumber(getDiskPath(diskPath))
		if err != nil {
			return "", errors.Wrapf(err, "failed to get disk ID")
		}
	}
	return diskID, nil
}

func (d *Disk) lvstoreToDisk(spdkClient *spdkclient.Client, lvstoreName, lvstoreUUID string) error {
	lvstores, err := spdkClient.BdevLvolGetLvstore(lvstoreName, lvstoreUUID)
	if err != nil {
		return errors.Wrapf(err, "failed to get lvstore with name %v and UUID %v", lvstoreName, lvstoreUUID)
	}
	lvstore := &lvstores[0]

	d.Name = lvstore.Name
	d.UUID = lvstore.UUID

	d.TotalSize = int64(lvstore.TotalDataClusters * lvstore.ClusterSize)
	d.FreeSize = int64(lvstore.FreeClusters * lvstore.ClusterSize)
	d.TotalBlocks = int64(lvstore.TotalDataClusters * lvstore.ClusterSize / lvstore.BlockSize)
	d.FreeBlocks = int64(lvstore.FreeClusters * lvstore.ClusterSize / lvstore.BlockSize)
	d.BlockSize = int64(lvstore.BlockSize)
	d.ClusterSize = int64(lvstore.ClusterSize)

	return nil
}

func (d *Disk) diskHealthGet(spdkClient *spdkclient.Client, diskName, diskDriver string) (*spdktypes.BdevNvmeControllerHealthInfo, error) {
	if diskName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "disk name is required")
	}

	// Currently, SPDK health info is only available for NVMe bdev controllers.
	if diskDriver != "" && diskDriver != string(commontypes.DiskDriverNvme) {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "disk driver %q does not support health info", diskDriver)
	}

	d.Lock()
	defer d.Unlock()

	healthInfo, err := spdkClient.BdevNvmeGetControllerHealthInfo(diskName)
	if err != nil {
		return nil, errors.Wrapf(err, "failed to get NVMe bdev controller health info for disk %q", diskName)
	}

	return &healthInfo, nil
}

func isNvmeDriver(diskDriver, diskPath string) bool {
	if diskDriver == string(commontypes.DiskDriverNvme) {
		return true
	}

	exactDiskDriver, err := spdkdisk.GetDiskDriver(commontypes.DiskDriver(diskDriver), diskPath)
	if err != nil {
		logrus.WithError(err).Warnf("failed to get disk driver for driver %v and path %s", diskDriver, diskPath)
		return false
	}

	return exactDiskDriver == commontypes.DiskDriverNvme
}
</file>

<file path="pkg/spdk/engine_test.go">
package spdk

//	. "gopkg.in/check.v1"

// func (s *TestSuite) xxTestcheckInitiatorAndTargetCreationRequirementsForNvmeTcpFrontend(c *C) {
// 	testCases := []struct {
// 		name                              string
// 		podIP                             string
// 		initiatorIP                       string
// 		targetIP                          string
// 		port                              int32
// 		targetPort                        int32
// 		standbyTargetPort                 int32
// 		expectedInitiatorCreationRequired bool
// 		expectedTargetCreationRequired    bool
// 		expectedError                     error
// 	}{
// 		{
// 			name:                              "Create both initiator and target instances",
// 			podIP:                             "192.168.1.1",
// 			initiatorIP:                       "192.168.1.1",
// 			targetIP:                          "192.168.1.1",
// 			port:                              0,
// 			targetPort:                        0,
// 			standbyTargetPort:                 0,
// 			expectedInitiatorCreationRequired: true,
// 			expectedTargetCreationRequired:    true,
// 			expectedError:                     nil,
// 		},
// 		{
// 			name:                              "Create local target instance on the node with initiator instance",
// 			podIP:                             "192.168.1.1",
// 			initiatorIP:                       "192.168.1.1",
// 			targetIP:                          "192.168.1.1",
// 			port:                              8080,
// 			targetPort:                        0,
// 			standbyTargetPort:                 0,
// 			expectedInitiatorCreationRequired: false,
// 			expectedTargetCreationRequired:    true,
// 			expectedError:                     nil,
// 		},
// 		{
// 			name:                              "Create local initiator instance only",
// 			podIP:                             "192.168.1.1",
// 			initiatorIP:                       "192.168.1.1",
// 			targetIP:                          "192.168.1.2",
// 			port:                              0,
// 			targetPort:                        0,
// 			standbyTargetPort:                 0,
// 			expectedInitiatorCreationRequired: true,
// 			expectedTargetCreationRequired:    false,
// 			expectedError:                     nil,
// 		},
// 		{
// 			name:                              "Create local target instance on the node without initiator instance",
// 			podIP:                             "192.168.1.2",
// 			initiatorIP:                       "192.168.1.1",
// 			targetIP:                          "192.168.1.2",
// 			port:                              0,
// 			targetPort:                        0,
// 			standbyTargetPort:                 0,
// 			expectedInitiatorCreationRequired: false,
// 			expectedTargetCreationRequired:    true,
// 			expectedError:                     nil,
// 		},
// 		{
// 			name:                              "Invalid initiator and target addresses",
// 			podIP:                             "192.168.1.1",
// 			initiatorIP:                       "192.168.1.2",
// 			targetIP:                          "192.168.1.3",
// 			port:                              0,
// 			targetPort:                        0,
// 			standbyTargetPort:                 0,
// 			expectedInitiatorCreationRequired: false,
// 			expectedTargetCreationRequired:    false,
// 			expectedError:                     fmt.Errorf("invalid initiator and target addresses for engine test-engine creation with initiator address 192.168.1.2 and target address 192.168.1.3"),
// 		},
// 		{
// 			name:                              "Standby target instance is already created",
// 			podIP:                             "192.168.1.1",
// 			initiatorIP:                       "192.168.1.1",
// 			targetIP:                          "192.168.1.1",
// 			port:                              100,
// 			targetPort:                        0,
// 			standbyTargetPort:                 105,
// 			expectedInitiatorCreationRequired: false,
// 			expectedTargetCreationRequired:    false,
// 			expectedError:                     nil,
// 		},
// 	}
// 	for testName, testCase := range testCases {
// 		c.Logf("testing checkInitiatorAndTargetCreationRequirementsForNvmeTcpFrontend.%v", testName)

// 		engine := &Engine{
// 			NvmeTcpFrontend: &NvmeTcpFrontend{
// 				Port:              testCase.port,
// 				TargetPort:        testCase.targetPort,
// 				StandbyTargetPort: testCase.standbyTargetPort,
// 			},

// 			Name: "test-engine",
// 			log:  safelog.NewSafeLogger(logrus.NewEntry(logrus.New())),
// 		}

// 		initiatorCreationRequired, targetCreationRequired, err := engine.checkInitiatorAndTargetCreationRequirements(testCase.podIP, testCase.initiatorIP, testCase.targetIP)

// 		c.Assert(initiatorCreationRequired, Equals, testCase.expectedInitiatorCreationRequired,
// 			Commentf("Test case '%s': unexpected initiator creation requirement", testCase.name))
// 		c.Assert(targetCreationRequired, Equals, testCase.expectedTargetCreationRequired,
// 			Commentf("Test case '%s': unexpected target creation requirement", testCase.name))
// 		c.Assert(err, DeepEquals, testCase.expectedError,
// 			Commentf("Test case '%s': unexpected error result", testCase.name))
// 	}
// }

// func (s *TestSuite) xxTestIsNewEngine(c *C) {
// 	testCases := []struct {
// 		name     string
// 		engine   *Engine
// 		expected bool
// 	}{
// 		{
// 			name:   "New engine with empty IP and TargetIP and StandbyTargetPort 0",
// 			engine: &Engine{
// 				// NvmeTcpFrontend: &NvmeTcpFrontend{
// 				// 	IP:                "",
// 				// 	TargetIP:          "",
// 				// 	StandbyTargetPort: 0,
// 				// },
// 			},
// 			expected: true,
// 		},
// 		{
// 			name:   "Engine with non-empty IP",
// 			engine: &Engine{
// 				// NvmeTcpFrontend: &NvmeTcpFrontend{
// 				// 	IP:                "192.168.1.1",
// 				// 	TargetIP:          "",
// 				// 	StandbyTargetPort: 0,
// 				// },
// 			},
// 			expected: false,
// 		},
// 		{
// 			name:   "Engine with non-empty TargetIP",
// 			engine: &Engine{
// 				// NvmeTcpFrontend: &NvmeTcpFrontend{
// 				// 	IP:                "",
// 				// 	TargetIP:          "192.168.1.2",
// 				// 	StandbyTargetPort: 0,
// 				// },
// 			},
// 			expected: false,
// 		},
// 		{
// 			name:   "Engine with non-zero StandbyTargetPort",
// 			engine: &Engine{
// 				// NvmeTcpFrontend: &NvmeTcpFrontend{
// 				// 	IP:                "",
// 				// 	TargetIP:          "",
// 				// 	StandbyTargetPort: 8080,
// 				// },
// 			},
// 			expected: false,
// 		},
// 	}

// 	for testName, testCase := range testCases {
// 		c.Logf("testing isNewEngine.%v", testName)
// 		result := testCase.engine.isNewNvmeTcpFrontendEngine()
// 		c.Assert(result, Equals, testCase.expected, Commentf("Test case '%s': unexpected result", testCase.name))
// 	}
// }

// func (s *TestSuite) xxTestReleaseTargetAndStandbyTargetPorts(c *C) {
// 	testCases := []struct {
// 		name                      string
// 		engine                    *Engine
// 		expectedTargetPort        int32
// 		expectedStandbyTargetPort int32
// 		expectedError             error
// 	}{
// 		{
// 			name: "Release both target and standby target ports",
// 			engine: &Engine{
// 				NvmeTcpFrontend: &NvmeTcpFrontend{
// 					TargetPort:        2000,
// 					StandbyTargetPort: 2005,
// 				},
// 			},
// 			expectedTargetPort:        0,
// 			expectedStandbyTargetPort: 0,
// 			expectedError:             nil,
// 		},
// 		{
// 			name: "Release target port only but standby target port is not set",
// 			engine: &Engine{
// 				NvmeTcpFrontend: &NvmeTcpFrontend{
// 					TargetPort:        2000,
// 					StandbyTargetPort: 0,
// 				},
// 			},
// 			expectedTargetPort:        0,
// 			expectedStandbyTargetPort: 0,
// 			expectedError:             nil,
// 		},
// 		{
// 			name: "Release target and standby ports when they are the same",
// 			engine: &Engine{
// 				NvmeTcpFrontend: &NvmeTcpFrontend{
// 					TargetPort:        2000,
// 					StandbyTargetPort: 2000,
// 				},
// 			},
// 			expectedTargetPort:        0,
// 			expectedStandbyTargetPort: 0,
// 			expectedError:             nil,
// 		},
// 		{
// 			name: "Release snapshot target port only",
// 			engine: &Engine{
// 				NvmeTcpFrontend: &NvmeTcpFrontend{
// 					TargetPort:        0,
// 					StandbyTargetPort: 2000,
// 				},
// 			},
// 			expectedTargetPort:        0,
// 			expectedStandbyTargetPort: 0,
// 			expectedError:             nil,
// 		},
// 	}

// 	for testName, testCase := range testCases {
// 		c.Logf("testing releaseTargetAndStandbyTargetPorts.%v", testName)

// 		bitmap, err := commonbitmap.NewBitmap(0, 100000)
// 		c.Assert(err, IsNil)

// 		err = testCase.engine.releaseTargetAndStandbyTargetPorts(bitmap)
// 		c.Assert(err, DeepEquals, testCase.expectedError, Commentf("Test case '%s': unexpected error result", testCase.name))
// 		c.Assert(testCase.engine.NvmeTcpFrontend.TargetPort, Equals, testCase.expectedTargetPort, Commentf("Test case '%s': unexpected target port", testCase.name))
// 		// c.Assert(testCase.engine.NvmeTcpFrontend.StandbyTargetPort, Equals, testCase.expectedStandbyTargetPort, Commentf("Test case '%s': unexpected standby target port", testCase.name))
// 	}
// }
</file>

<file path="pkg/spdk/engine.go">
package spdk

import (
	"fmt"
	"net"
	"strconv"
	"strings"
	"sync"
	"time"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"k8s.io/apimachinery/pkg/util/wait"
	"k8s.io/client-go/util/retry"

	retrygo "github.com/avast/retry-go/v4"
	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/backupstore"
	"github.com/longhorn/go-spdk-helper/pkg/jsonrpc"
	"github.com/longhorn/types/pkg/generated/spdkrpc"

	commonbitmap "github.com/longhorn/go-common-libs/bitmap"
	commonnet "github.com/longhorn/go-common-libs/net"
	commonutils "github.com/longhorn/go-common-libs/utils"
	spdkclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
	spdktypes "github.com/longhorn/go-spdk-helper/pkg/spdk/types"
	helpertypes "github.com/longhorn/go-spdk-helper/pkg/types"
	helperutil "github.com/longhorn/go-spdk-helper/pkg/util"

	"github.com/longhorn/longhorn-spdk-engine/pkg/api"
	"github.com/longhorn/longhorn-spdk-engine/pkg/client"
	"github.com/longhorn/longhorn-spdk-engine/pkg/types"
	"github.com/longhorn/longhorn-spdk-engine/pkg/util"

	safelog "github.com/longhorn/longhorn-spdk-engine/pkg/log"
)

type NvmeTcpTarget struct {
	IP   string
	Port int32

	Nqn   string
	Nguid string
}

// ReplicaAdder abstracts the two pluggable steps of the replica-add flow:
// shallow copy and finish. Production code uses realReplicaAdder; tests
// can supply a MockReplicaAdder via Engine.SetReplicaAdder().
type ReplicaAdder interface {
	ReplicaShallowCopy(dstReplicaServiceCli *client.SPDKClient, srcReplicaName, dstReplicaName string, rebuildingSnapshots []*api.Lvol, fastSync bool) error
	ReplicaAddFinish(srcReplicaServiceCli, dstReplicaServiceCli *client.SPDKClient, srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress string) error
}

// realReplicaAdder is the production ReplicaAdder that delegates to Engine methods.
type realReplicaAdder struct {
	e *Engine
}

func (ra *realReplicaAdder) ReplicaShallowCopy(dstReplicaServiceCli *client.SPDKClient, srcReplicaName, dstReplicaName string, rebuildingSnapshots []*api.Lvol, fastSync bool) error {
	return ra.e.replicaShallowCopy(dstReplicaServiceCli, srcReplicaName, dstReplicaName, rebuildingSnapshots, fastSync)
}

func (ra *realReplicaAdder) ReplicaAddFinish(srcReplicaServiceCli, dstReplicaServiceCli *client.SPDKClient, srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress string) error {
	return ra.e.replicaAddFinish(srcReplicaServiceCli, dstReplicaServiceCli, srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress)
}

// MockReplicaAdder allows tests to override specific replica-add operations.
// Set individual function fields to non-nil to mock that operation; nil fields
// fall through to the real Engine implementation via the embedded Real adder.
//
// When a mock FinishFunc injects an error, it is the mock's responsibility
// to call Real.ReplicaAddFinish() for SPDK resource cleanup before returning
// the error. The engine goroutine will NOT perform fallback cleanup.
type MockReplicaAdder struct {
	Real            ReplicaAdder
	ShallowCopyFunc func(dstReplicaServiceCli *client.SPDKClient, srcReplicaName, dstReplicaName string, rebuildingSnapshots []*api.Lvol, fastSync bool) error
	FinishFunc      func(srcReplicaServiceCli, dstReplicaServiceCli *client.SPDKClient, srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress string) error
}

func (m *MockReplicaAdder) ReplicaShallowCopy(dstReplicaServiceCli *client.SPDKClient, srcReplicaName, dstReplicaName string, rebuildingSnapshots []*api.Lvol, fastSync bool) error {
	if m.ShallowCopyFunc != nil {
		return m.ShallowCopyFunc(dstReplicaServiceCli, srcReplicaName, dstReplicaName, rebuildingSnapshots, fastSync)
	}
	return m.Real.ReplicaShallowCopy(dstReplicaServiceCli, srcReplicaName, dstReplicaName, rebuildingSnapshots, fastSync)
}

func (m *MockReplicaAdder) ReplicaAddFinish(srcReplicaServiceCli, dstReplicaServiceCli *client.SPDKClient, srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress string) error {
	if m.FinishFunc != nil {
		return m.FinishFunc(srcReplicaServiceCli, dstReplicaServiceCli, srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress)
	}
	return m.Real.ReplicaAddFinish(srcReplicaServiceCli, dstReplicaServiceCli, srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress)
}

type Engine struct {
	sync.RWMutex

	Name       string
	VolumeName string
	SpecSize   uint64
	ActualSize uint64
	Frontend   string
	Endpoint   string

	ctrlrLossTimeout     int
	fastIOFailTimeoutSec int
	ReplicaStatusMap     map[string]*EngineReplicaStatus

	RaidBdevUUID string

	NvmeTcpTarget *NvmeTcpTarget

	State    types.InstanceState
	ErrorMsg string

	Head        *api.Lvol
	SnapshotMap map[string]*api.Lvol

	IsRestoring           bool
	RestoringSnapshotName string

	isExpanding           bool
	lastExpansionFailedAt string
	lastExpansionError    string

	// UpdateCh should not be protected by the engine lock
	UpdateCh chan interface{}

	log *safelog.SafeLogger

	// replicaAdder provides the pluggable replica-add operations (shallow copy + finish).
	// Production uses realReplicaAdder; tests can supply MockReplicaAdder.
	replicaAdder ReplicaAdder

	// replicaAddFinishUnlockedHook is an optional hook that fires inside
	// replicaAddFinish after the Engine lock is released and before the slow
	// RPC calls (ReplicaRebuildingSrcFinish / ReplicaRebuildingDstFinish).
	//
	// Purpose: regression guard for the 3-phase lock pattern. The hook lets
	// tests call TryLock() to prove the lock is truly released during phase 2.
	// Without this, a future change that accidentally holds the lock through
	// the RPCs (reverting to single-phase) would be undetectable from
	// external behavior alone — replica-add would still succeed or fail
	// identically, but all other Engine operations would stall for 10+
	// seconds on same-node NVMe-oF ETIMEDOUT.
	replicaAddFinishUnlockedHook func()
}

type EngineReplicaStatus struct {
	Address  string
	BdevName string
	Mode     types.Mode
}

func NewEngine(engineName, volumeName, frontend string, specSize uint64, engineUpdateCh chan interface{}) *Engine {
	log := logrus.StandardLogger().WithFields(logrus.Fields{
		"engineName": engineName,
		"volumeName": volumeName,
	})

	roundedSpecSize := util.RoundUp(specSize, helpertypes.MiB)
	if roundedSpecSize != specSize {
		log.Infof("Rounded up spec size from %v to %v since the spec size should be multiple of MiB", specSize, roundedSpecSize)
	}
	log.WithField("specSize", roundedSpecSize)

	e := &Engine{
		Name:       engineName,
		VolumeName: volumeName,
		Frontend:   frontend,
		SpecSize:   specSize,

		// TODO: support user-defined values
		ctrlrLossTimeout:     replicaCtrlrLossTimeoutSec,
		fastIOFailTimeoutSec: replicaFastIOFailTimeoutSec,

		ReplicaStatusMap: map[string]*EngineReplicaStatus{},

		NvmeTcpTarget: &NvmeTcpTarget{},

		State: types.InstanceStatePending,

		SnapshotMap: map[string]*api.Lvol{},

		UpdateCh: engineUpdateCh,

		log: safelog.NewSafeLogger(log),
	}
	e.replicaAdder = &realReplicaAdder{e: e}
	return e
}

func (e *Engine) Create(spdkClient *spdkclient.Client, replicaAddressMap map[string]string, portCount int32, superiorPortAllocator *commonbitmap.Bitmap,
	salvageRequested bool) (ret *spdkrpc.Engine, err error) {
	e.log.WithFields(logrus.Fields{
		"portCount":         portCount,
		"replicaAddressMap": replicaAddressMap,
		"salvageRequested":  salvageRequested,
		"frontend":          e.Frontend,
	}).Info("Creating engine")

	requireUpdate := true

	e.Lock()
	defer func() {
		e.Unlock()
		if requireUpdate {
			e.UpdateCh <- nil
		}
	}()

	if e.State != types.InstanceStatePending {
		requireUpdate = false
		return nil, fmt.Errorf("invalid state %s for engine %s creation", e.State, e.Name)
	}

	if err := e.validateReplicaSize(replicaAddressMap); err != nil {
		return nil, errors.Wrapf(err, "failed to validate replica size during engine target creation")
	}

	defer func() {
		if err != nil {
			e.log.WithError(err).Errorf("Failed to create engine %s", e.Name)
			if e.State != types.InstanceStateError {
				e.State = types.InstanceStateError
			}
			e.ErrorMsg = err.Error()

			ret = e.getWithoutLock()
			err = nil
		} else {
			if e.State != types.InstanceStateError {
				e.ErrorMsg = ""
			}
		}
	}()

	_, err = spdkClient.BdevRaidGet(e.Name, 0)
	if err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return nil, errors.Wrapf(err, "failed to get raid bdev %v during engine creation", e.Name)
	}

	if salvageRequested {
		e.log.Info("Requesting salvage for engine replicas")
		replicaAddressMap, err = e.filterSalvageCandidates(replicaAddressMap)
		if err != nil {
			return nil, errors.Wrapf(err, "failed to update replica mode to filter salvage candidates")
		}
	}

	replicaBdevList := []string{}
	for replicaName, replicaAddr := range replicaAddressMap {
		e.ReplicaStatusMap[replicaName] = &EngineReplicaStatus{
			Address: replicaAddr,
		}

		bdevName, err := connectNVMfBdev(spdkClient, replicaName, replicaAddr, e.ctrlrLossTimeout, e.fastIOFailTimeoutSec, maxRetries, retryInterval)
		if err != nil {
			e.log.WithError(err).Warnf("Failed to get bdev from replica %s with address %s during engine creation, will mark the mode to ERR and continue", replicaName, replicaAddr)
			e.ReplicaStatusMap[replicaName].Mode = types.ModeERR
		} else {
			// TODO: Check if a replica is really a RW replica rather than a rebuilding failed replica
			e.ReplicaStatusMap[replicaName].Mode = types.ModeRW
			e.ReplicaStatusMap[replicaName].BdevName = bdevName
			replicaBdevList = append(replicaBdevList, bdevName)
		}
	}

	e.log.UpdateLoggerWithWarnOnFailure(logrus.Fields{
		"replicaStatusMap": e.ReplicaStatusMap,
	}, "Failed to update logger with replica status map during engine creation")

	e.checkAndUpdateInfoFromReplicaNoLock()

	e.log.Infof("Connecting all available replicas %+v, then launching raid during engine creation", e.ReplicaStatusMap)
	if _, err := spdkClient.BdevRaidCreate(e.Name, spdktypes.BdevRaidLevel1, 0, replicaBdevList, ""); err != nil {
		return nil, err
	}

	switch e.Frontend {
	case types.FrontendSPDKTCPBlockdev, types.FrontendSPDKTCPNvmf:
		e.log.Infof("Creating NVMe TCP target for engine %v", e.Name)
		if err := e.createNVMeTCPTarget(spdkClient, superiorPortAllocator, portCount); err != nil {
			return nil, errors.Wrapf(err, "failed to create NVMe TCP target for engine %v", e.Name)
		}
	case types.FrontendUBLK:
		e.log.Infof("Creating UBLK target for engine %v", e.Name)
		if err := spdkClient.UblkCreateTarget("", true); err != nil {
			return nil, err
		}
	}

	e.State = types.InstanceStateRunning

	e.log.Info("Created engine target")

	return e.getWithoutLock(), nil
}

func (e *Engine) createNVMeTCPTarget(spdkClient *spdkclient.Client, superiorPortAllocator *commonbitmap.Bitmap, portCount int32) error {
	podIP, err := commonnet.GetIPForPod()
	if err != nil {
		return err
	}

	port, _, err := superiorPortAllocator.AllocateRange(portCount)
	if err != nil {
		return errors.Wrapf(err, "failed to allocate port for engine target %v", e.Name)
	}

	e.NvmeTcpTarget.IP = podIP
	e.NvmeTcpTarget.Port = port
	e.NvmeTcpTarget.Nguid = generateNGUID(e.Name)
	e.NvmeTcpTarget.Nqn = helpertypes.GetNQN(e.Name)

	e.log.Info("Blindly stopping expose RAID bdev for engine")
	if err := spdkClient.StopExposeBdev(e.NvmeTcpTarget.Nqn); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return errors.Wrapf(err, "failed to blindly stop exposing RAID bdev for engine target %v", e.Name)
	}

	e.log.Infof("Starting to expose RAID bdev for engine target %v on %v:%v", e.Name, e.NvmeTcpTarget.IP, e.NvmeTcpTarget.Port)
	if err := spdkClient.StartExposeBdev(e.NvmeTcpTarget.Nqn, e.Name, e.NvmeTcpTarget.Nguid,
		e.NvmeTcpTarget.IP, strconv.Itoa(int(e.NvmeTcpTarget.Port))); err != nil {
		// No need to release ports here. The engine will be marked as ERR by
		// Create's deferred error handler, and Delete will release the ports
		// when the user cleans up this engine.
		return errors.Wrapf(err, "failed to start exposing RAID bdev for engine target %v", e.Name)
	}

	return nil
}

func (e *Engine) validateReplicaSize(replicaAddressMap map[string]string) error {
	if len(replicaAddressMap) == 0 {
		return fmt.Errorf("no replicas provided for engine %s", e.Name)
	}

	// Validate the engine & replica sizes before creating the engine
	replicaSizeMap := make(map[string]uint64, len(replicaAddressMap))
	for replicaName, replicaAddr := range replicaAddressMap {
		replicaClient, err := GetServiceClient(replicaAddr)
		if err != nil {
			return err
		}
		replica, err := replicaClient.ReplicaGet(replicaName)
		if err != nil {
			return errors.Wrapf(err, "failed to get replica %v from %v", replicaName, replicaAddr)
		}

		replicaSizeMap[replicaName] = replica.SpecSize
	}

	// check if all replica sizes are the same
	expectedSize := uint64(0)
	for _, replicaSize := range replicaSizeMap {
		if expectedSize == 0 {
			expectedSize = replicaSize
			continue
		}

		if expectedSize != replicaSize {
			return fmt.Errorf("found different replica sizes: %+v", replicaSizeMap)
		}
	}

	if e.SpecSize < expectedSize {
		return fmt.Errorf("engine spec size %d is smaller than replica size %d", e.SpecSize, expectedSize)
	}

	return nil
}

// filterSalvageCandidates updates the replicaAddressMap by retaining only replicas
// eligible for salvage based on the largest volume head size.
//
// It iterates through all replicas and:
//   - Retrieves the volume head size for each replica.
//   - Identifies replicas with the largest volume head size as salvage candidates.
//   - Remove the replicas that are not eligible as salvage candidates.
func (e *Engine) filterSalvageCandidates(replicaAddressMap map[string]string) (map[string]string, error) {
	// Initialize filteredCandidates to hold a copy of replicaAddressMap.
	filteredCandidates := map[string]string{}
	for key, value := range replicaAddressMap {
		filteredCandidates[key] = value
	}

	volumeHeadSizeToReplicaNames := map[uint64][]string{}

	// Collect volume head size for each replica.
	for replicaName, replicaAddress := range replicaAddressMap {
		func() {
			// Get service client for the current replica.
			replicaServiceCli, err := GetServiceClient(replicaAddress)
			if err != nil {
				e.log.WithError(err).Warnf("Skipping salvage for replica %s with address %s due to failed to get replica service client", replicaName, replicaAddress)
				return
			}

			defer func() {
				if errClose := replicaServiceCli.Close(); errClose != nil {
					e.log.WithError(errClose).Errorf("Failed to close replica %s client with address %s during salvage candidate filtering", replicaName, replicaAddress)
				}
			}()

			// Retrieve replica information.
			replica, err := replicaServiceCli.ReplicaGet(replicaName)
			if err != nil {
				e.log.WithError(err).Warnf("Skipping salvage for replica %s with address %s due to failed to get replica info", replicaName, replicaAddress)
				delete(filteredCandidates, replicaName)
				return
			}

			// Map volume head size to replica names.
			volumeHeadSizeToReplicaNames[replica.Head.ActualSize] = append(volumeHeadSizeToReplicaNames[replica.Head.ActualSize], replicaName)
		}()
	}

	// Sort the volume head sizes to find the largest.
	volumeHeadSizeSorted, err := commonutils.SortKeys(volumeHeadSizeToReplicaNames)
	if err != nil {
		return nil, errors.Wrap(err, "failed to sort keys of salvage candidate by volume head size")
	}

	if len(volumeHeadSizeSorted) == 0 {
		return nil, errors.New("failed to find any salvage candidate with volume head size")
	}

	// Determine salvage candidates with the largest volume head size.
	largestVolumeHeadSize := volumeHeadSizeSorted[len(volumeHeadSizeSorted)-1]
	e.log.Infof("Selecting salvage candidates with the largest volume head size %v from %+v", largestVolumeHeadSize, volumeHeadSizeToReplicaNames)

	// Filter out replicas that do not match the largest volume head size.
	salvageCandidates := volumeHeadSizeToReplicaNames[largestVolumeHeadSize]
	for replicaName := range replicaAddressMap {
		if !commonutils.Contains(salvageCandidates, replicaName) {
			e.log.Infof("Skipping salvage for replica %s with address %s due to not having the largest volume head size (%v)", replicaName, replicaAddressMap[replicaName])
			delete(filteredCandidates, replicaName)
			continue
		}

		e.log.Infof("Including replica %s as a salvage candidate", replicaName)
	}

	return filteredCandidates, nil
}

func (e *Engine) Delete(spdkClient *spdkclient.Client, superiorPortAllocator *commonbitmap.Bitmap) (err error) {
	requireUpdate := false

	e.Lock()
	defer func() {
		// Considering that there may be still pending validations, it's better to update the state after the deletion.
		if err != nil {
			e.log.WithError(err).Errorf("Failed to delete engine %s", e.Name)
			if e.State != types.InstanceStateError {
				e.State = types.InstanceStateError
				e.ErrorMsg = err.Error()
				e.log.WithError(err).Error("Failed to delete engine")
				requireUpdate = true
			}
		} else {
			if e.State != types.InstanceStateError {
				e.ErrorMsg = ""
			}
		}
		if e.State == types.InstanceStateRunning {
			e.State = types.InstanceStateTerminating
			requireUpdate = true
		}

		e.Unlock()

		if requireUpdate {
			e.UpdateCh <- nil
		}
	}()

	e.log.Info("Deleting engine")

	e.log.Infof("Stopping to expose RAID bdev for engine %s", e.Name)
	switch e.Frontend {
	case types.FrontendUBLK:
		if err := spdkClient.UblkDestroyTarget(); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return errors.Wrapf(err, "failed to destroy UBLK target for engine %s", e.Name)
		}
	case types.FrontendSPDKTCPBlockdev, types.FrontendSPDKTCPNvmf:
		if err := spdkClient.StopExposeBdev(e.NvmeTcpTarget.Nqn); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return errors.Wrapf(err, "failed to stop exposing bdev for engine %s", e.Name)
		}
	}

	if e.NvmeTcpTarget != nil {
		e.NvmeTcpTarget.Nqn = ""
		e.NvmeTcpTarget.Nguid = ""
		e.NvmeTcpTarget.IP = ""
		// Port is released by releasePorts below.
	}

	// Release the ports if they are allocated
	if err := e.releasePorts(superiorPortAllocator); err != nil {
		return err
	}

	requireUpdate = true

	if _, err := spdkClient.BdevRaidDelete(e.Name); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return err
	}

	requireUpdate, err = e.disconnectReplicas(spdkClient)
	if err != nil {
		return err
	}

	e.log.Info("Deleted engine")

	return nil
}

func (e *Engine) disconnectReplicas(spdkClient *spdkclient.Client) (requireUpdate bool, err error) {
	for replicaName, replicaStatus := range e.ReplicaStatusMap {
		if err := disconnectNVMfBdev(spdkClient, replicaStatus.BdevName, disconnectMaxRetries, disconnectRetryInterval); err != nil {
			if replicaStatus.Mode != types.ModeERR {
				e.log.WithError(err).Errorf("Engine failed to disconnect replica %s with bdev %s during deletion, will update the mode from %v to ERR", replicaName, replicaStatus.BdevName, replicaStatus.Mode)
				replicaStatus.Mode = types.ModeERR
				requireUpdate = true
			}
			return requireUpdate, err
		}
		delete(e.ReplicaStatusMap, replicaName)
		requireUpdate = true
	}

	return requireUpdate, nil
}

func (e *Engine) releasePorts(superiorPortAllocator *commonbitmap.Bitmap) error {
	if e.NvmeTcpTarget == nil {
		return nil
	}

	err := releasePortIfExists(superiorPortAllocator,
		map[int32]struct{}{
			e.NvmeTcpTarget.Port: {},
		},
		e.NvmeTcpTarget.Port)

	e.NvmeTcpTarget.Port = 0

	return err
}

func releasePortIfExists(superiorPortAllocator *commonbitmap.Bitmap, ports map[int32]struct{}, port int32) error {
	if port == 0 {
		return nil
	}

	_, exists := ports[port]
	if exists {
		if err := superiorPortAllocator.ReleaseRange(port, port); err != nil {
			return err
		}
		delete(ports, port)
	}

	return nil
}

func (e *Engine) Get() (res *spdkrpc.Engine) {
	e.RLock()
	defer e.RUnlock()

	return e.getWithoutLock()
}

func (e *Engine) getWithoutLock() (res *spdkrpc.Engine) {
	res = &spdkrpc.Engine{
		Name:                  e.Name,
		VolumeName:            e.VolumeName,
		SpecSize:              e.SpecSize,
		ActualSize:            e.ActualSize,
		ReplicaAddressMap:     map[string]string{},
		ReplicaModeMap:        map[string]spdkrpc.ReplicaMode{},
		Snapshots:             map[string]*spdkrpc.Lvol{},
		Frontend:              e.Frontend,
		Endpoint:              e.Endpoint,
		State:                 string(e.State),
		ErrorMsg:              e.ErrorMsg,
		IsExpanding:           e.isExpanding,
		LastExpansionError:    e.lastExpansionError,
		LastExpansionFailedAt: e.lastExpansionFailedAt,
	}

	if e.NvmeTcpTarget != nil {
		res.Ip = e.NvmeTcpTarget.IP
		res.Port = e.NvmeTcpTarget.Port
	}

	for replicaName, replicaStatus := range e.ReplicaStatusMap {
		res.ReplicaAddressMap[replicaName] = replicaStatus.Address
		res.ReplicaModeMap[replicaName] = types.ReplicaModeToGRPCReplicaMode(replicaStatus.Mode)
	}

	res.Head = api.LvolToProtoLvol(e.Head)

	for snapshotName, snapApiLvol := range e.SnapshotMap {
		res.Snapshots[snapshotName] = api.LvolToProtoLvol(snapApiLvol)
	}

	return res
}

type replicaAddFrontendSuspendResumeWrapper func(work func() error) error

// ReplicaAdd performs the full replica-add flow consisting of three phases:
//
// Phase 0 — Synchronous Setup (under Engine lock, returns on completion):
//  1. Validate engine state is Running, dst replica doesn't exist, no other WO replica.
//  2. Obtain replica gRPC clients for all existing replicas, plus src/dst rebuild clients.
//  3. Pick an RW replica as the rebuild source.
//  4. Call replicaAddStart (optionally wrapped by frontendSuspendResumeWrapper for EF suspend/resume):
//     a. Create rebuild snapshot across all replicas.
//     b. Get rebuilding snapshot list from src replica.
//     c. ReplicaRebuildingSrcStart: src replica exposes snapshot as NVMe-oF target.
//     d. ReplicaRebuildingDstStart: dst replica attaches external snapshot, creates head.
//     e. BdevRaidGrowBaseBdev: add dst head bdev to RAID as base bdev.
//     f. Mark dst replica as WO in ReplicaStatusMap.
//  5. Launch replicaAddAsync goroutine (Phase 1–2 below).
//  6. Return nil (or setupErr on failure) — background goroutine takes over.
//     On sync error: outer defer marks dst replica ERR and (if applicable) sets engine to Error state.
//
// Phase 1 — Shallow Copy (replicaAddAsync goroutine):
//  7. Check for setup errors: if Phase 0 failed (setupErr != nil), set asyncErr and skip to cleanup.
//  8. adder.ReplicaShallowCopy(): copy all snapshots from src to dst; on failure set asyncErr.
//
// Phase 2 — Finish or Cleanup (replicaAddCleanupOrFinish, two mutually exclusive paths):
//
//	Path A — Failure (asyncErr != nil):
//	  9. Call e.replicaAddFinish() directly (no frontendSuspendResumeWrapper, no suspend/resume) for SPDK resource cleanup.
//	     Replica is already ERR. replicaAddFinish uses the same DstFinish→SrcFinish order as the success path.
//
//	Path B — Success (asyncErr == nil):
//	 10. Call adder.ReplicaAddFinish() via frontendSuspendResumeWrapper (if present) or directly.
//	     frontendSuspendResumeWrapper (buildGRPCReplicaAddFrontendSuspendResumeWrapper) does:
//	       a. EF Suspend (gRPC to EngineFrontend).
//	       b. Execute replicaAddFinish (3-phase lock pattern):
//	          Phase 1 (lock): read dst mode. Phase 2 (unlock): RPC calls. Phase 3 (lock): update mode.
//	       c. EF Resume (gRPC to EngineFrontend).
//	 11. If finish returns error: mark dst replica ERR. SPDK resource cleanup is NOT retried —
//	     it is the responsibility of the ReplicaAdder (mock should call Real.ReplicaAddFinish()
//	     before returning error) or r.Delete() when the replica is subsequently removed.
func (e *Engine) ReplicaAdd(spdkClient *spdkclient.Client, dstReplicaName, dstReplicaAddress string, fastSync bool, frontendSuspendResumeWrapper replicaAddFrontendSuspendResumeWrapper) (err error) {
	updateRequired := false

	e.Lock()
	defer func() {
		e.Unlock()

		if updateRequired {
			e.UpdateCh <- nil
		}
	}()

	e.log.Infof("Engine is starting replica %s add", dstReplicaName)

	// Syncing with the SPDK TGT server only when the engine is running.
	if e.State != types.InstanceStateRunning {
		return fmt.Errorf("invalid state %v for engine %s replica %s add start", e.State, e.Name, dstReplicaName)
	}

	if _, exists := e.ReplicaStatusMap[dstReplicaName]; exists {
		return fmt.Errorf("replica %s already exists", dstReplicaName)
	}

	for replicaName, replicaStatus := range e.ReplicaStatusMap {
		if replicaStatus.Mode == types.ModeWO {
			return fmt.Errorf("cannot add a new replica %s since there is already a rebuilding replica %s", dstReplicaName, replicaName)
		}
	}

	// engineErr will be set when the engine failed to do any non-recoverable operations, then there is no way to make the engine continue working. Typically, it's related to the frontend suspend or resume failures.
	// While err means replica-related operation errors. It will fail the current replica add flow.
	var engineErr error
	var srcReplicaName, srcReplicaAddress string
	var srcReplicaServiceCli, dstReplicaServiceCli *client.SPDKClient

	defer func() {
		if engineErr != nil {
			if e.State != types.InstanceStateError {
				e.State = types.InstanceStateError
				updateRequired = true
			}
			e.ErrorMsg = engineErr.Error()
		} else {
			if e.State != types.InstanceStateError {
				e.ErrorMsg = ""
			}
		}
		if engineErr != nil || err != nil {
			prevMode := types.Mode("")
			if e.ReplicaStatusMap[dstReplicaName] != nil {
				prevMode = e.ReplicaStatusMap[dstReplicaName].Mode
				e.ReplicaStatusMap[dstReplicaName].Mode = types.ModeERR
				e.ReplicaStatusMap[dstReplicaName].Address = dstReplicaAddress
			} else {
				e.ReplicaStatusMap[dstReplicaName] = &EngineReplicaStatus{
					Mode:    types.ModeERR,
					Address: dstReplicaAddress,
				}
			}

			e.log.WithError(err).Errorf("Engine failed to start replica %s rebuilding, will mark the rebuilding replica mode from %v to ERR", dstReplicaName, prevMode)
			updateRequired = true
		}
	}()

	replicaClients, err := e.getReplicaClients()
	if err != nil {
		return err
	}
	defer e.closeReplicaClients(replicaClients)

	srcReplicaName, srcReplicaAddress, err = e.getReplicaAddSrcReplica()
	if err != nil {
		return err
	}

	// On error, getSrcAndDstReplicaClients() closes any partially created
	// replica service clients before returning. On success, these clients are
	// intentionally kept open here and are closed later by replicaAddFinish(),
	// which is reached from either the async cleanup path or the async finish path.
	srcReplicaServiceCli, dstReplicaServiceCli, err = e.getSrcAndDstReplicaClients(srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress)
	if err != nil {
		return err
	}

	// Perform the synchronous setup phase (optionally wrapped for EF suspend/resume).
	var rebuildingSnapshotList []*api.Lvol
	var setupErr error
	startFn := func() error {
		var startEngineErr error
		var startUpdateRequired bool
		rebuildingSnapshotList, startUpdateRequired, startEngineErr, setupErr = e.replicaAddStart(spdkClient, replicaClients,
			srcReplicaServiceCli, dstReplicaServiceCli, srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress)
		updateRequired = updateRequired || startUpdateRequired
		if startEngineErr != nil {
			engineErr = startEngineErr
		}
		if setupErr != nil {
			return setupErr
		}
		return startEngineErr
	}

	if frontendSuspendResumeWrapper != nil {
		if wrapErr := frontendSuspendResumeWrapper(startFn); wrapErr != nil {
			// The wrapper itself may fail (e.g. suspend or resume failure).
			// If the inner startFn already set engineErr, those are captured
			// via closure.
			if err == nil && engineErr == nil {
				engineErr = wrapErr
			}
			setupErr = wrapErr
		}
	} else {
		if fnErr := startFn(); fnErr != nil {
			setupErr = fnErr
		}
	}

	// Launch the async phase: shallow copy followed by cleanup or finish.
	// Even on setup failure, the goroutine handles SPDK resource cleanup
	// (exposed snapshot, NVMe connections) via replicaAddCleanupOrFinish.
	go e.replicaAddAsync(srcReplicaServiceCli, dstReplicaServiceCli, srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress, rebuildingSnapshotList, fastSync, frontendSuspendResumeWrapper, setupErr)

	if setupErr != nil {
		return setupErr
	}

	// TODO: Mark the destination replica as WO mode here does not prevent the RAID bdev from using this. May need to have a SPDK API to control the corresponding base bdev mode.
	// Reading data from this dst replica is not a good choice as the flow will be more zigzag than reading directly from the src replica:
	// application -> RAID1 -> this base bdev (dest replica) -> the exposed snapshot (src replica).
	e.log.UpdateLoggerWithWarnOnFailure(logrus.Fields{
		"replicaStatusMap": e.ReplicaStatusMap,
	}, "Failed to update logger with replica status map during engine creation")

	e.log.Infof("Engine started to rebuild replica %s from healthy replica %s with fastSync %v", dstReplicaName, srcReplicaName, fastSync)

	return nil
}

// replicaAddStart performs the synchronous setup phase of replica add:
// creates a rebuild snapshot, starts src/dst rebuilding, connects the
// dst head bdev, and grows the RAID base bdev.
//
// Returns:
//   - rebuildingSnapshotList: snapshots to be shallow-copied in the async phase
//   - startUpdateRequired: true if engine state changed and UpdateCh should be notified
//   - engineErr: non-nil if the engine should transition to Error state
//   - err: non-nil for replica-related operation errors
func (e *Engine) replicaAddStart(spdkClient *spdkclient.Client, replicaClients map[string]*client.SPDKClient,
	srcReplicaServiceCli, dstReplicaServiceCli *client.SPDKClient,
	srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress string,
) (rebuildingSnapshotList []*api.Lvol, startUpdateRequired bool, engineErr, err error) {
	snapshotName := GenerateRebuildingSnapshotName()
	opts := &api.SnapshotOptions{
		Timestamp: util.Now(),
	}

	var replicasErr error
	startUpdateRequired, replicasErr, engineErr = e.snapshotOperationWithoutLock(spdkClient, replicaClients, snapshotName, SnapshotOperationCreate, opts)
	if replicasErr != nil {
		return nil, startUpdateRequired, engineErr, replicasErr
	}
	if engineErr != nil {
		return nil, startUpdateRequired, engineErr, nil
	}
	e.checkAndUpdateInfoFromReplicaNoLock()

	rebuildingSnapshotList, err = getRebuildingSnapshotList(srcReplicaServiceCli, srcReplicaName)
	if err != nil {
		return nil, startUpdateRequired, nil, err
	}

	// Ask the source replica to expose the newly created snapshot if the source replica and destination replica are not on the same node.
	externalSnapshotAddress, err := srcReplicaServiceCli.ReplicaRebuildingSrcStart(srcReplicaName, dstReplicaName, dstReplicaAddress, snapshotName)
	if err != nil {
		return nil, startUpdateRequired, nil, err
	}

	// The destination replica attaches the source replica exposed snapshot as the external snapshot then create a head based on it.
	dstHeadLvolAddress, err := dstReplicaServiceCli.ReplicaRebuildingDstStart(dstReplicaName, srcReplicaName, srcReplicaAddress, snapshotName, externalSnapshotAddress, rebuildingSnapshotList)
	if err != nil {
		return nil, startUpdateRequired, nil, err
	}

	// Add rebuilding replica head bdev to the base bdev list of the RAID bdev
	dstHeadLvolBdevName, err := connectNVMfBdev(spdkClient, dstReplicaName, dstHeadLvolAddress, e.ctrlrLossTimeout, e.fastIOFailTimeoutSec, maxRetries, retryInterval)
	if err != nil {
		return nil, startUpdateRequired, nil, err
	}

	e.log.Infof("Adding rebuilding replica %s head bdev %s to the base bdev list for engine %s", dstReplicaName, dstHeadLvolBdevName, e.Name)
	if _, err := spdkClient.BdevRaidGrowBaseBdev(e.Name, dstHeadLvolBdevName); err != nil {
		return nil, startUpdateRequired, nil, errors.Wrapf(err, "failed to adding the rebuilding replica %s head bdev %s to the base bdev list for engine %s", dstReplicaName, dstHeadLvolBdevName, e.Name)
	}

	e.ReplicaStatusMap[dstReplicaName] = &EngineReplicaStatus{
		Address:  dstReplicaAddress,
		Mode:     types.ModeWO,
		BdevName: dstHeadLvolBdevName,
	}
	startUpdateRequired = true
	return rebuildingSnapshotList, startUpdateRequired, nil, nil
}

// replicaAddAsync runs the asynchronous phase of replica add: shallow copy
// followed by cleanup or finish. It is launched as a goroutine from ReplicaAdd.
// setupErr is non-nil if the synchronous setup phase (replicaAddStart) failed.
func (e *Engine) replicaAddAsync(
	srcReplicaServiceCli, dstReplicaServiceCli *client.SPDKClient,
	srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress string,
	rebuildingSnapshotList []*api.Lvol,
	fastSync bool,
	frontendSuspendResumeWrapper replicaAddFrontendSuspendResumeWrapper,
	setupErr error,
) {
	defer func() {
		if r := recover(); r != nil {
			e.log.Errorf("Recovered panic during engine %s replica %s add: %v", e.Name, dstReplicaName, r)
		}
	}()

	// Resolve the replica adder under lock
	e.RLock()
	adder := e.replicaAdder
	e.RUnlock()

	var asyncErr error
	defer func() {
		e.replicaAddCleanupOrFinish(adder, srcReplicaServiceCli, dstReplicaServiceCli, srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress, frontendSuspendResumeWrapper, asyncErr)
	}()

	// Check for errors from the synchronous setup phase
	if setupErr != nil {
		asyncErr = fmt.Errorf("replica add setup failed: %v", setupErr)
		e.log.Errorf("Engine %s won't do shallow copy for replica %s add due to setup error: %v", e.Name, dstReplicaName, setupErr)
		return
	}

	// Shallow copy phase
	if scErr := adder.ReplicaShallowCopy(dstReplicaServiceCli, srcReplicaName, dstReplicaName, rebuildingSnapshotList, fastSync); scErr != nil {
		asyncErr = scErr
		e.log.WithError(scErr).Errorf("Engine %s failed to do the shallow copy for replica %s add", e.Name, dstReplicaName)
		return
	}
}

// replicaAddCleanupOrFinish handles the completion of the async replica add phase.
// If asyncErr is non-nil, it calls replicaAddFinish directly (no frontendSuspendResumeWrapper) for
// SPDK resource cleanup. If asyncErr is nil, it runs the finish flow via the adder
// (optionally wrapped by frontendSuspendResumeWrapper for suspend/resume).
func (e *Engine) replicaAddCleanupOrFinish(
	adder ReplicaAdder,
	srcReplicaServiceCli, dstReplicaServiceCli *client.SPDKClient,
	srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress string,
	frontendSuspendResumeWrapper replicaAddFrontendSuspendResumeWrapper,
	asyncErr error,
) {
	if asyncErr != nil {
		// Setup or shallow copy failed. The replica is already
		// marked ERR by replicaShallowCopy's defer or ReplicaAdd's
		// outer defer. Call replicaAddFinish directly (no wrapper)
		// for SPDK resource cleanup (exposed snapshot, NVMe
		// connections). The mode is already ERR, so replicaAddFinish
		// uses the same DstFinish→SrcFinish order as the success path.
		if cleanupErr := e.replicaAddFinish(srcReplicaServiceCli, dstReplicaServiceCli, srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress); cleanupErr != nil {
			e.log.WithError(cleanupErr).Errorf("Engine %s failed to clean up after replica %s add failure", e.Name, dstReplicaName)
		}
		return
	}

	// Success path: shallow copy completed, run finish.
	e.log.Infof("Starting to finish replica %s add for engine %s", dstReplicaName, e.Name)

	finishFn := func() error {
		return adder.ReplicaAddFinish(srcReplicaServiceCli, dstReplicaServiceCli, srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress)
	}

	var finishErr error
	if frontendSuspendResumeWrapper != nil {
		finishErr = frontendSuspendResumeWrapper(finishFn)
	} else {
		finishErr = finishFn()
	}
	if finishErr != nil {
		e.log.WithError(finishErr).Errorf("Engine %s failed to finish replica %s add", e.Name, dstReplicaName)

		// Mark the replica as ERR. SPDK resource cleanup is the
		// responsibility of the ReplicaAdder (mock should call
		// Real.ReplicaAddFinish before returning error) or will
		// be handled by r.Delete() when the replica is removed.
		e.Lock()
		if dstStatus := e.ReplicaStatusMap[dstReplicaName]; dstStatus != nil && dstStatus.Mode != types.ModeERR {
			dstStatus.Mode = types.ModeERR
		}
		e.Unlock()
	}
}

func (e *Engine) closeReplicaAddClients(srcReplicaServiceCli, dstReplicaServiceCli *client.SPDKClient, srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress, phase string) {
	if srcReplicaServiceCli != nil {
		if errClose := srcReplicaServiceCli.Close(); errClose != nil {
			e.log.WithError(errClose).Errorf("Engine %s failed to close source replica %s client with address %s during %s", e.Name, srcReplicaName, srcReplicaAddress, phase)
		}
	}
	if dstReplicaServiceCli != nil {
		if errClose := dstReplicaServiceCli.Close(); errClose != nil {
			e.log.WithError(errClose).Errorf("Engine %s failed to close dest replica %s client with address %s during %s", e.Name, dstReplicaName, dstReplicaAddress, phase)
		}
	}
}

func (e *Engine) getSrcAndDstReplicaClients(srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress string) (srcReplicaServiceCli, dstReplicaServiceCli *client.SPDKClient, err error) {
	defer func() {
		if err != nil {
			if srcReplicaServiceCli != nil {
				if errClose := srcReplicaServiceCli.Close(); errClose != nil {
					e.log.WithError(errClose).Errorf("Failed to close source replica %s client with address %s during get get src and dst replica clients", srcReplicaName, srcReplicaAddress)
				}
			}
			if dstReplicaServiceCli != nil {
				if errClose := dstReplicaServiceCli.Close(); errClose != nil {
					e.log.WithError(errClose).Errorf("Failed to close dest replica %s client with address %s during get get src and dst replica clients", dstReplicaName, dstReplicaAddress)
				}
			}
			srcReplicaServiceCli = nil
			dstReplicaServiceCli = nil
		}
	}()

	srcReplicaServiceCli, err = GetServiceClient(srcReplicaAddress)
	if err != nil {
		return
	}
	dstReplicaServiceCli, err = GetServiceClient(dstReplicaAddress)
	return
}

func (e *Engine) replicaShallowCopy(dstReplicaServiceCli *client.SPDKClient, srcReplicaName, dstReplicaName string, rebuildingSnapshotList []*api.Lvol, fastSync bool) (err error) {
	updateRequired := false
	defer func() {
		if updateRequired {
			e.UpdateCh <- nil
		}
	}()

	defer func() {
		// Blindly mark the rebuilding replica as mode ERR now.
		if err != nil {
			e.Lock()
			if e.ReplicaStatusMap[dstReplicaName] != nil && e.ReplicaStatusMap[dstReplicaName].Mode != types.ModeERR {
				e.log.WithError(err).Errorf("Engine failed to do shallow copy from src replica %s to dst replica %s, will mark the rebuilding replica mode from %v to ERR", srcReplicaName, dstReplicaName, e.ReplicaStatusMap[dstReplicaName].Mode)
				e.ReplicaStatusMap[dstReplicaName].Mode = types.ModeERR
				updateRequired = true
			}
			e.Unlock()
		}
	}()

	e.log.Infof("Engine is starting snapshots shallow copy from rebuilding src replica %s to rebuilding dst replica %s", srcReplicaName, dstReplicaName)

	rebuildingSnapshotMap := map[string]*api.Lvol{}
	for _, snapshotApiLvol := range rebuildingSnapshotList {
		rebuildingSnapshotMap[snapshotApiLvol.Name] = snapshotApiLvol
	}

	// Traverse the src replica snapshot tree with a DFS way and do shallow copy one by one
	timer := time.NewTimer(MaxShallowCopyWaitTime)
	defer timer.Stop()
	ticker := time.NewTicker(ShallowCopyCheckInterval)
	defer ticker.Stop()
	currentSnapshotName := ""
	for idx := 0; idx < len(rebuildingSnapshotList); idx++ {
		currentSnapshotName = rebuildingSnapshotList[idx].Name
		e.log.Infof("Engine is syncing snapshot %s from rebuilding src replica %s to rebuilding dst replica %s", currentSnapshotName, srcReplicaName, dstReplicaName)

		if err := dstReplicaServiceCli.ReplicaRebuildingDstShallowCopyStart(dstReplicaName, currentSnapshotName, fastSync); err != nil {
			return errors.Wrapf(err, "failed to start shallow copy snapshot %s", currentSnapshotName)
		}

		timer.Reset(MaxShallowCopyWaitTime)
		continuousRetryCount := 0
		for finished := false; !finished; {
			select {
			case <-timer.C:
				return errors.Errorf("Timeout engine failed to check the dst replica %s snapshot %s shallow copy status over %d times", dstReplicaName, currentSnapshotName, maxRetries)
			case <-ticker.C:
				shallowCopyStatus, err := dstReplicaServiceCli.ReplicaRebuildingDstShallowCopyCheck(dstReplicaName)
				if err != nil {
					continuousRetryCount++
					if continuousRetryCount > maxRetries {
						return errors.Wrapf(err, "Engine failed to check the dst replica %s snapshot %s shallow copy status over %d times", dstReplicaName, currentSnapshotName, maxRetries)
					}
					e.log.WithError(err).Errorf("Engine failed to check the dst replica %s snapshot %s shallow copy status, retry count %d", dstReplicaName, currentSnapshotName, continuousRetryCount)
					continue
				}
				if shallowCopyStatus.State == helpertypes.ShallowCopyStateError || shallowCopyStatus.Error != "" {
					return fmt.Errorf("rebuilding error during shallow copy for snapshot %s: %s", shallowCopyStatus.SnapshotName, shallowCopyStatus.Error)
				}

				continuousRetryCount = 0
				if shallowCopyStatus.State == helpertypes.ShallowCopyStateComplete {
					if shallowCopyStatus.Progress != 100 {
						e.log.Warnf("Shallow copy snapshot %s is %s but somehow the progress is not 100%%", shallowCopyStatus.SnapshotName, helpertypes.ShallowCopyStateComplete)
					}
					e.log.Infof("Shallow copied snapshot %s", shallowCopyStatus.SnapshotName)
					finished = true
					break // nolint: staticcheck
				}
			}
		}

		snapshotOptions := &api.SnapshotOptions{
			UserCreated: rebuildingSnapshotMap[currentSnapshotName].UserCreated,
			Timestamp:   rebuildingSnapshotMap[currentSnapshotName].SnapshotTimestamp,
		}

		if err = dstReplicaServiceCli.ReplicaRebuildingDstSnapshotCreate(dstReplicaName, currentSnapshotName, snapshotOptions); err != nil {
			return err
		}
	}

	e.log.Infof("Engine shallow copied all snapshots from rebuilding src replica %s to rebuilding dst replica %s", srcReplicaName, dstReplicaName)

	return nil
}

// replicaAddFinish tries its best to finish the replica add no matter if the dst replica is rebuilt successfully or not.
// It returns fatal errors that lead to engine unavailable only. As for the errors during replica rebuilding wrap-up, it will be logged and ignored.
//
// The function uses a 3-phase lock pattern to avoid holding the Engine lock during
// potentially slow RPC calls (ReplicaRebuildingSrcFinish, ReplicaRebuildingDstFinish):
//
//	Phase 1 (lock):   Read dst replica mode from ReplicaStatusMap
//	Phase 2 (unlock): Execute RPC calls (DstFinish → SrcFinish) to src/dst replicas
//	Phase 3 (lock):   Update replica mode and engine state
func (e *Engine) replicaAddFinish(srcReplicaServiceCli, dstReplicaServiceCli *client.SPDKClient, srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress string) error {
	defer e.closeReplicaAddClients(srcReplicaServiceCli, dstReplicaServiceCli,
		srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress, "add replica finish")

	// Phase 1: Read replica state under lock
	e.Lock()
	dstReplicaStatus := e.ReplicaStatusMap[dstReplicaName]
	var dstMode types.Mode
	if dstReplicaStatus != nil {
		dstMode = dstReplicaStatus.Mode
	}
	e.Unlock()

	// Phase 2: Execute RPC calls without holding the Engine lock.
	// These calls may be slow (e.g. bdev_nvme_detach_controller returning ETIMEDOUT).
	// By releasing the lock, other Engine operations (status queries, other replica
	// operations) are not blocked during these potentially slow RPCs.
	e.RLock()
	phase2Hook := e.replicaAddFinishUnlockedHook
	e.RUnlock()
	if phase2Hook != nil {
		phase2Hook()
	}

	// The RPC order is always DstFinish first, then SrcFinish.
	// DstFinish calls BdevLvolSetParent (for ModeWO) to switch the dst
	// replica's snapshot chain from the external (src-exposed) snapshot
	// to the locally rebuilt chain. This parent-switch requires the
	// external snapshot bdev to still be accessible, so the src must
	// keep exposing it until DstFinish completes. For ModeERR the parent
	// switch is a no-op, but we still call DstFinish to actively clean
	// up dst-side resources (external snapshot attachment, NVMe controller).
	var dstReplicaErr error
	if dstReplicaStatus == nil {
		// Dst replica was already removed from the engine map during Phase 1→2.
		// Skip dst-side finish, but still clean up src-side resources (exposed
		// snapshot, NVMe-oF target, port) so they don't leak.
		e.log.Infof("Engine skipped finishing rebuilding dst replica %s as it was already removed, will still clean up src replica %s", dstReplicaName, srcReplicaName)
		if srcReplicaServiceCli != nil {
			if srcErr := srcReplicaServiceCli.ReplicaRebuildingSrcFinish(srcReplicaName, dstReplicaName); srcErr != nil {
				// WARNING: src replica may retain residual rebuilding state
				// (exposed snapshot, NVMe-oF target, port, dstRebuildingBdevName)
				// that will block subsequent rebuilds using this src replica.
				// Because dst replica is already removed from the engine map,
				// there is no engine-side state to mark ERR.
				// The residual state will be cleaned up when the src replica is
				// itself deleted (Replica.Delete calls doCleanupForRebuildingSrc).
				e.log.WithError(srcErr).Errorf("Engine failed to finish rebuilding src replica %s after dst replica %s was removed: src may retain residual rebuilding state (exposed snapshot, port) until src replica is deleted", srcReplicaName, dstReplicaName)
			}
		}
	} else if srcReplicaServiceCli == nil || dstReplicaServiceCli == nil {
		// The clients can be nil when replicaAddFinish is called for cleanup
		// after an early failure in ReplicaAdd (e.g. getReplicaClients or
		// getReplicaAddSrcReplica failed before clients were created).
		// Skip RPC calls; Phase 3 will still update the replica mode.
		e.log.Warnf("Engine skipping rebuilding RPC cleanup for replica %s because replica service clients are unavailable (src=%v, dst=%v)", dstReplicaName, srcReplicaServiceCli != nil, dstReplicaServiceCli != nil)
	} else {
		// Unified path: DstFinish first, then SrcFinish
		if dstErr := dstReplicaServiceCli.ReplicaRebuildingDstFinish(dstReplicaName); dstErr != nil {
			e.log.WithError(dstErr).Errorf("Engine failed to finish rebuilding dst replica %s, will update the mode from %v to ERR then continue rebuilding src replica %s finish", dstReplicaName, dstMode, srcReplicaName)
			dstReplicaErr = dstErr
		}

		// The source replica blindly stops exposing the snapshot and wipes
		// the rebuilding info. If this fails, the src replica retains residual
		// rebuilding state (exposed snapshot, NVMe-oF target, port,
		// dstRebuildingBdevName) that will be cleaned up when the src replica
		// is itself deleted (doCleanupForRebuildingSrc). This does NOT block
		// dst promotion to RW because the dst data is already correct after
		// a successful DstFinish (parent switch completed).
		if srcErr := srcReplicaServiceCli.ReplicaRebuildingSrcFinish(srcReplicaName, dstReplicaName); srcErr != nil {
			// TODO: Should we mark this healthy replica as error?
			e.log.WithError(srcErr).Errorf("Engine failed to finish rebuilding src replica %s, will ignore this error", srcReplicaName)
		}

		if dstReplicaErr == nil && dstMode == types.ModeWO {
			e.log.Infof("Engine succeeded to finish rebuilding dst replica %s, will update the mode from %v to RW", dstReplicaName, dstMode)
		}
	}

	// Phase 3: Update engine state under lock
	updateRequired := false

	e.Lock()
	defer func() {
		e.Unlock()

		if updateRequired {
			e.UpdateCh <- nil
		}
	}()

	if e.State != types.InstanceStateError {
		e.ErrorMsg = ""
	}

	// Re-read replica status — it may have been removed while we were unlocked.
	// Use the current mode (not the phase-1 snapshot dstMode) to decide the
	// state transition, so that concurrent downgrades (e.g. validateReplicaStatusMapNoLock
	// setting WO → ERR during unlocked phase 2) are not overwritten with RW.
	dstReplicaStatus = e.ReplicaStatusMap[dstReplicaName]
	if dstReplicaStatus != nil {
		switch dstReplicaStatus.Mode {
		case types.ModeERR:
			updateRequired = true
		case types.ModeWO:
			if dstReplicaErr != nil {
				dstReplicaStatus.Mode = types.ModeERR
			} else {
				dstReplicaStatus.Mode = types.ModeRW
			}
			updateRequired = true
		}
	}

	e.checkAndUpdateInfoFromReplicaNoLock()

	if dstReplicaErr != nil {
		e.log.Errorf("Engine failed to finish rebuilding replica %s from healthy replica %s (dstErr=%v)", dstReplicaName, srcReplicaName, dstReplicaErr)
	} else if dstReplicaStatus != nil && dstReplicaStatus.Mode == types.ModeERR {
		// All RPCs succeeded, but the replica mode is ERR because another
		// goroutine (e.g. validateReplicaStatusMapNoLock) downgraded WO → ERR
		// during the unlocked phase 2. Phase 3 correctly preserved that
		// concurrent downgrade instead of overwriting it with RW.
		e.log.Warnf("Engine finished rebuilding RPC cleanup for replica %s from healthy replica %s, but replica mode is ERR due to concurrent downgrade during unlocked phase", dstReplicaName, srcReplicaName)
	} else {
		e.log.Infof("Engine finished rebuilding replica %s from healthy replica %s", dstReplicaName, srcReplicaName)
	}

	return nil
}

// getReplicaAddSrcReplica picks the first RW replica from ReplicaStatusMap as the rebuild source.
func (e *Engine) getReplicaAddSrcReplica() (srcReplicaName, srcReplicaAddress string, err error) {
	for replicaName, replicaStatus := range e.ReplicaStatusMap {
		if replicaStatus.Mode != types.ModeRW {
			continue
		}
		srcReplicaName = replicaName
		srcReplicaAddress = replicaStatus.Address
		break
	}
	if srcReplicaName == "" || srcReplicaAddress == "" {
		return "", "", fmt.Errorf("cannot find an RW replica in engine %s during replica add", e.Name)
	}
	return srcReplicaName, srcReplicaAddress, nil
}

// getRebuildingSnapshotList fetches the snapshot tree from the src replica and
// returns the ordered list of snapshots that need to be shallow-copied to the
// dst replica. It finds the ancestor snapshot (empty parent or backing image
// parent) and traverses the tree via DFS to produce the copy order.
func getRebuildingSnapshotList(srcReplicaServiceCli *client.SPDKClient, srcReplicaName string) ([]*api.Lvol, error) {
	rpcSrcReplica, err := srcReplicaServiceCli.ReplicaGet(srcReplicaName)
	if err != nil {
		return []*api.Lvol{}, err
	}
	ancestorSnapshotName, latestSnapshotName := "", ""
	for snapshotName, snapApiLvol := range rpcSrcReplica.Snapshots {
		// If the parent is empty, it's the ancestor snapshot
		// Notice that the ancestor snapshot parent is still empty even if there is a backing image
		if snapApiLvol.Parent == "" || types.IsBackingImageSnapLvolName(snapApiLvol.Parent) {
			ancestorSnapshotName = snapshotName
		}
		if snapApiLvol.Children[types.VolumeHead] {
			latestSnapshotName = snapshotName
		}
	}
	if ancestorSnapshotName == "" || latestSnapshotName == "" {
		return []*api.Lvol{}, fmt.Errorf("cannot find the ancestor snapshot %s or latest snapshot %s from RW replica %s snapshot map during engine replica add", ancestorSnapshotName, latestSnapshotName, srcReplicaName)
	}

	return retrieveRebuildingSnapshotList(rpcSrcReplica, ancestorSnapshotName, []*api.Lvol{}), nil
}

// retrieveRebuildingSnapshotList recursively traverses the replica snapshot tree with a DFS way
func retrieveRebuildingSnapshotList(rpcSrcReplica *api.Replica, currentSnapshotName string, rebuildingSnapshotList []*api.Lvol) []*api.Lvol {
	if currentSnapshotName == "" || currentSnapshotName == types.VolumeHead {
		return rebuildingSnapshotList
	}
	rebuildingSnapshotList = append(rebuildingSnapshotList, rpcSrcReplica.Snapshots[currentSnapshotName])
	for childSnapshotName := range rpcSrcReplica.Snapshots[currentSnapshotName].Children {
		rebuildingSnapshotList = retrieveRebuildingSnapshotList(rpcSrcReplica, childSnapshotName, rebuildingSnapshotList)
	}
	return rebuildingSnapshotList
}

func (e *Engine) ReplicaDelete(spdkClient *spdkclient.Client, replicaName, replicaAddress string) (err error) {
	e.log.Infof("Deleting replica %s with address %s from engine", replicaName, replicaAddress)

	e.Lock()
	defer e.Unlock()

	if replicaName == "" {
		for rName, rStatus := range e.ReplicaStatusMap {
			if rStatus.Address == replicaAddress {
				replicaName = rName
				break
			}
		}
	}
	if replicaName == "" {
		return fmt.Errorf("cannot find replica name with address %s for engine %s replica delete", replicaAddress, e.Name)
	}
	replicaStatus := e.ReplicaStatusMap[replicaName]
	if replicaStatus == nil {
		return fmt.Errorf("cannot find replica %s from the replica status map for engine %s replica delete", replicaName, e.Name)
	}
	if replicaAddress != "" && replicaStatus.Address != replicaAddress {
		return fmt.Errorf("replica %s recorded address %s does not match the input address %s for engine %s replica delete", replicaName, replicaStatus.Address, replicaAddress, e.Name)
	}

	e.log.Infof("Removing base bdev %v from engine", replicaStatus.BdevName)
	if _, err := spdkClient.BdevRaidRemoveBaseBdev(replicaStatus.BdevName); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return errors.Wrapf(err, "failed to remove base bdev %s for deleting replica %s", replicaStatus.BdevName, replicaName)
	}

	controllerName := helperutil.GetNvmeControllerNameFromNamespaceName(replicaStatus.BdevName)
	// Fallback to use replica name. Make sure there won't be a leftover controller even if somehow `replicaStatus.BdevName` has no record
	if controllerName == "" {
		e.log.Infof("No NVMf controller found for replica %s, so fallback to use replica name %s", replicaName, replicaName)
		controllerName = replicaName
	}
	// Detaching the corresponding NVMf controller to remote replica
	e.log.Infof("Detaching the corresponding NVMf controller %v during remote replica %s delete", controllerName, replicaName)
	if _, err := spdkClient.BdevNvmeDetachController(controllerName); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return errors.Wrapf(err, "failed to detach controller %s for deleting replica %s", controllerName, replicaName)
	}

	delete(e.ReplicaStatusMap, replicaName)

	e.log.UpdateLoggerWithWarnOnFailure(logrus.Fields{
		"replicaStatusMap": e.ReplicaStatusMap,
	}, "Failed to update logger with replica status map during engine creation")

	return nil
}

type SnapshotOperationType string

const (
	SnapshotOperationCreate = SnapshotOperationType("snapshot-create")
	SnapshotOperationDelete = SnapshotOperationType("snapshot-delete")
	SnapshotOperationRevert = SnapshotOperationType("snapshot-revert")
	SnapshotOperationPurge  = SnapshotOperationType("snapshot-purge")
	SnapshotOperationHash   = SnapshotOperationType("snapshot-hash")
)

func (e *Engine) SnapshotCreate(spdkClient *spdkclient.Client, inputSnapshotName string) (snapshotName string, err error) {
	e.log.Infof("Creating snapshot %s", inputSnapshotName)

	opts := &api.SnapshotOptions{
		UserCreated: true,
		Timestamp:   util.Now(),
	}

	return e.snapshotOperation(spdkClient, inputSnapshotName, SnapshotOperationCreate, opts)
}

func (e *Engine) SnapshotDelete(spdkClient *spdkclient.Client, snapshotName string) (err error) {
	e.log.Infof("Deleting snapshot %s", snapshotName)

	_, err = e.snapshotOperation(spdkClient, snapshotName, SnapshotOperationDelete, nil)
	return err
}

func (e *Engine) SnapshotRevert(spdkClient *spdkclient.Client, snapshotName string) (err error) {
	e.log.Infof("Reverting snapshot %s", snapshotName)

	_, err = e.snapshotOperation(spdkClient, snapshotName, SnapshotOperationRevert, nil)
	return err
}

func (e *Engine) SnapshotPurge(spdkClient *spdkclient.Client) (err error) {
	e.log.Infof("Purging snapshots")

	_, err = e.snapshotOperation(spdkClient, "", SnapshotOperationPurge, nil)
	return err
}

func (e *Engine) SnapshotHash(spdkClient *spdkclient.Client, snapshotName string, rehash bool) (err error) {
	e.log.Infof("Hashing snapshot %s, rehash %v", snapshotName, rehash)

	_, err = e.snapshotOperation(spdkClient, snapshotName, SnapshotOperationHash, rehash)
	return err
}

func (e *Engine) snapshotOperation(spdkClient *spdkclient.Client, inputSnapshotName string, snapshotOp SnapshotOperationType, opts any) (snapshotName string, err error) {
	updateRequired := false

	e.Lock()
	defer func() {
		e.Unlock()

		if updateRequired {
			e.UpdateCh <- nil
		}
	}()

	// Syncing with the SPDK TGT server only when the engine is running.
	if e.State != types.InstanceStateRunning {
		return "", fmt.Errorf("invalid state %v for engine %s snapshot %s operation", e.State, e.Name, inputSnapshotName)
	}

	replicaClients, err := e.getReplicaClients()
	if err != nil {
		return "", err
	}
	defer e.closeReplicaClients(replicaClients)

	if snapshotName, err = e.snapshotOperationPreCheckWithoutLock(replicaClients, inputSnapshotName, snapshotOp); err != nil {
		return "", err
	}

	var engineErr, replicasErr error
	defer func() {
		if engineErr != nil {
			if e.State != types.InstanceStateError {
				e.State = types.InstanceStateError
				updateRequired = true
			}
			e.ErrorMsg = engineErr.Error()
		} else {
			if e.State != types.InstanceStateError {
				e.ErrorMsg = ""
			}
		}
	}()

	updateRequired, replicasErr, engineErr = e.snapshotOperationWithoutLock(spdkClient, replicaClients, snapshotName, snapshotOp, opts)
	if replicasErr != nil {
		return "", replicasErr
	}
	if engineErr != nil {
		return "", engineErr
	}

	e.checkAndUpdateInfoFromReplicaNoLock()

	e.log.Infof("Engine finished snapshot operation %s name %s", snapshotOp, snapshotName)

	return snapshotName, nil
}

func (e *Engine) getReplicaClients() (replicaClients map[string]*client.SPDKClient, err error) {
	replicaClients = map[string]*client.SPDKClient{}
	for replicaName, replicaStatus := range e.ReplicaStatusMap {
		if replicaStatus.Mode != types.ModeRW && replicaStatus.Mode != types.ModeWO {
			continue
		}
		if replicaStatus.Address == "" {
			continue
		}
		c, err := GetServiceClient(replicaStatus.Address)
		if err != nil {
			return nil, err
		}
		replicaClients[replicaName] = c
	}

	return replicaClients, nil
}

func (e *Engine) closeReplicaClients(replicaClients map[string]*client.SPDKClient) {
	for replicaName := range replicaClients {
		if replicaClients[replicaName] != nil {
			if errClose := replicaClients[replicaName].Close(); errClose != nil {
				e.log.WithError(errClose).Errorf("Failed to close replica %s client", replicaName)
			}
		}
	}
}

func (e *Engine) snapshotOperationPreCheckWithoutLock(replicaClients map[string]*client.SPDKClient, snapshotName string, snapshotOp SnapshotOperationType) (string, error) {
	if snapshotOp == SnapshotOperationCreate && snapshotName == "" {
		snapshotName = util.UUID()[:8]
	}

	if snapshotOp == SnapshotOperationDelete {
		if snapshotName == "" {
			return "", fmt.Errorf("empty snapshot name for engine %s snapshot deletion", e.Name)
		}
		// Refresh snapshot topology before validation to avoid stale SnapshotMap checks.
		e.checkAndUpdateInfoFromReplicaNoLock()
		if e.SnapshotMap[snapshotName] == nil {
			return "", fmt.Errorf("engine %s does not contain snapshot %s during snapshot deletion", e.Name, snapshotName)
		}
		if len(e.SnapshotMap[snapshotName].Children) > 1 {
			return "", fmt.Errorf("engine %s cannot delete snapshot %s since it contains multiple children %+v", e.Name, snapshotName, e.SnapshotMap[snapshotName].Children)
		}
	}

	for replicaName := range replicaClients {
		replicaStatus := e.ReplicaStatusMap[replicaName]
		if replicaStatus == nil {
			return "", fmt.Errorf("cannot find replica %s in the engine %s replica status map before snapshot %s operation", replicaName, e.Name, snapshotName)
		}
		switch snapshotOp {
		case SnapshotOperationCreate:
		case SnapshotOperationDelete:
			if replicaStatus.Mode == types.ModeWO {
				return "", fmt.Errorf("engine %s contains WO replica %s during snapshot %s delete", e.Name, replicaName, snapshotName)
			}
		case SnapshotOperationRevert:
			if snapshotName == "" {
				return "", fmt.Errorf("empty snapshot name for engine %s snapshot deletion", e.Name)
			}
			if e.Frontend != types.FrontendEmpty {
				return "", fmt.Errorf("invalid frontend %v for engine %s snapshot %s revert", e.Frontend, e.Name, snapshotName)
			}
			if replicaStatus.Mode == types.ModeWO {
				return "", fmt.Errorf("engine %s contains WO replica %s during snapshot %s revert", e.Name, replicaName, snapshotName)
			}
			r, err := replicaClients[replicaName].ReplicaGet(replicaName)
			if err != nil {
				return "", err
			}
			if r.Snapshots[snapshotName] == nil {
				return "", fmt.Errorf("replica %s does not contain the reverting snapshot %s", replicaName, snapshotName)
			}
		case SnapshotOperationPurge:
			if replicaStatus.Mode == types.ModeWO {
				return "", fmt.Errorf("engine %s contains WO replica %s during snapshot purge", e.Name, replicaName)
			}
			// TODO: Do we need to verify that all replicas hold the same system snapshot list?
		case SnapshotOperationHash:
			if replicaStatus.Mode == types.ModeWO {
				return "", fmt.Errorf("engine %s contains WO replica %s during snapshot hash", e.Name, replicaName)
			}
			// TODO: Do we need to verify that all replicas hold the same system snapshot list?
		default:
			return "", fmt.Errorf("unknown replica snapshot operation %s", snapshotOp)
		}
	}

	return snapshotName, nil
}

func (e *Engine) snapshotOperationWithoutLock(spdkClient *spdkclient.Client, replicaClients map[string]*client.SPDKClient, snapshotName string, snapshotOp SnapshotOperationType, opts any) (updated bool, replicasErr error, engineErr error) {
	if snapshotOp == SnapshotOperationRevert {
		if _, err := spdkClient.BdevRaidDelete(e.Name); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			e.log.WithError(err).Errorf("Failed to delete RAID after snapshot %s revert", snapshotName)
			return false, err, err
		}
	}

	replicaErrorList := []error{}
	for replicaName := range replicaClients {
		replicaStatus := e.ReplicaStatusMap[replicaName]
		if replicaStatus == nil {
			return false, fmt.Errorf("cannot find replica %s in the engine %s replica status map during snapshot %s operation", replicaName, e.Name, snapshotName), nil
		}
		if err := e.replicaSnapshotOperation(spdkClient, replicaClients[replicaName], replicaName, snapshotName, snapshotOp, opts); err != nil && replicaStatus.Mode != types.ModeERR {
			replicaErrorList = append(replicaErrorList, err)
			if snapshotOp != SnapshotOperationHash {
				e.log.WithError(err).Errorf("Engine failed to issue operation %s for replica %s snapshot %s, will mark the replica mode from %v to ERR", snapshotOp, replicaName, snapshotName, replicaStatus.Mode)
				replicaStatus.Mode = types.ModeERR
				updated = true
			}
		}
	}
	replicasErr = util.CombineErrors(replicaErrorList...)

	if snapshotOp == SnapshotOperationRevert {
		replicaBdevList := []string{}
		for _, replicaStatus := range e.ReplicaStatusMap {
			if replicaStatus.Mode != types.ModeRW {
				continue
			}
			if replicaStatus.BdevName == "" {
				continue
			}
			replicaBdevList = append(replicaBdevList, replicaStatus.BdevName)
		}

		engineErr = retrygo.Do(
			func() error {
				_, err := spdkClient.BdevRaidCreate(e.Name, spdktypes.BdevRaidLevel1, 0, replicaBdevList, "")
				return err
			},
			retrygo.Attempts(uint(maxRetries)),
			retrygo.Delay(retryInterval),
			retrygo.LastErrorOnly(true),
		)
	}

	return updated, replicasErr, engineErr
}

func (e *Engine) replicaSnapshotOperation(spdkClient *spdkclient.Client, replicaClient *client.SPDKClient, replicaName, snapshotName string, snapshotOp SnapshotOperationType, opts any) error {
	switch snapshotOp {
	case SnapshotOperationCreate:
		// TODO: execute `sync` for the NVMe initiator before snapshot start
		optsPtr, ok := opts.(*api.SnapshotOptions)
		if !ok {
			return fmt.Errorf("invalid opts types %+v for snapshot create operation", opts)
		}
		return replicaClient.ReplicaSnapshotCreate(replicaName, snapshotName, optsPtr)
	case SnapshotOperationDelete:
		return replicaClient.ReplicaSnapshotDelete(replicaName, snapshotName)
	case SnapshotOperationRevert:
		replicaStatus := e.ReplicaStatusMap[replicaName]
		if replicaStatus == nil {
			return fmt.Errorf("cannot find replica %s in the engine %s replica status map during snapshot %s operation", replicaName, e.Name, snapshotName)
		}
		if err := disconnectNVMfBdev(spdkClient, replicaStatus.BdevName, disconnectMaxRetries, disconnectRetryInterval); err != nil {
			return err
		}
		replicaStatus.BdevName = ""
		// If the below step failed, the replica will be marked as ERR during ValidateAndUpdate.
		if err := replicaClient.ReplicaSnapshotRevert(replicaName, snapshotName); err != nil {
			return err
		}
		bdevName, err := connectNVMfBdev(spdkClient, replicaName, replicaStatus.Address, e.ctrlrLossTimeout, e.fastIOFailTimeoutSec, maxRetries, retryInterval)
		if err != nil {
			return err
		}
		if bdevName != "" {
			replicaStatus.BdevName = bdevName
		}
	case SnapshotOperationPurge:
		return replicaClient.ReplicaSnapshotPurge(replicaName)
	case SnapshotOperationHash:
		rehash, ok := opts.(bool)
		if !ok {
			return fmt.Errorf("rehash should be a boolean value for snapshot hash operation")
		}
		if err := replicaClient.ReplicaSnapshotHash(replicaName, snapshotName, rehash); err != nil {
			return err
		}
	default:
		return fmt.Errorf("unknown replica snapshot operation %s", snapshotOp)
	}

	return nil
}

func (e *Engine) SnapshotHashStatus(snapshotName string) (*spdkrpc.EngineSnapshotHashStatusResponse, error) {
	resp := &spdkrpc.EngineSnapshotHashStatusResponse{
		Status: map[string]*spdkrpc.ReplicaSnapshotHashStatusResponse{},
	}

	e.Lock()
	defer e.Unlock()

	for replicaName, replicaStatus := range e.ReplicaStatusMap {
		if replicaStatus.Mode != types.ModeRW {
			continue
		}

		replicaSnapshotHashStatusResponse, err := e.getReplicaSnapshotHashStatus(replicaName, replicaStatus.Address, snapshotName)
		if err != nil {
			return nil, err
		}
		resp.Status[replicaStatus.Address] = replicaSnapshotHashStatusResponse
	}

	return resp, nil
}

type replicaCandidate struct {
	ip      string
	lvsUUID string
	address string
}

func (e *Engine) SnapshotClone(snapshotName, srcEngineName, srcEngineAddress string, cloneMode spdkrpc.CloneMode) (err error) {
	e.Lock()
	defer e.Unlock()

	defer func() {
		err = errors.Wrap(err, "failed to do SnapshotClone")
	}()

	e.log.Infof("Engine is starting cloning snapshot %s", snapshotName)

	if len(e.ReplicaStatusMap) != 1 {
		return fmt.Errorf("destination engine must only have 1 replica when doing snapshot clone. Current "+
			"replica count is %v", len(e.ReplicaStatusMap))
	}

	dstReplicaName, dstReplicaAddr := "", ""
	for rName, rStatus := range e.ReplicaStatusMap {
		if rStatus.Mode != types.ModeRW {
			continue
		}
		dstReplicaName = rName
		dstReplicaAddr = rStatus.Address
		break
	}

	if dstReplicaName == "" || dstReplicaAddr == "" {
		return fmt.Errorf("cannot find a RW destination replica")
	}

	e.log.Infof("Selecting replica %v with address %v as dst replica for cloning", dstReplicaName, dstReplicaAddr)

	dstReplicaServiceCli, err := GetServiceClient(dstReplicaAddr)
	if err != nil {
		return err
	}
	defer func() {
		if errClose := dstReplicaServiceCli.Close(); errClose != nil {
			e.log.WithError(errClose).Errorf("Engine %v failed to close dst replica %v client with address %v",
				e.Name, dstReplicaName, dstReplicaAddr)
		}
	}()

	dstReplica, err := dstReplicaServiceCli.ReplicaGet(dstReplicaName)
	if err != nil {
		return err
	}

	srcEngineServiceCli, err := GetServiceClient(srcEngineAddress)
	if err != nil {
		return err
	}
	defer func() {
		if errClose := srcEngineServiceCli.Close(); errClose != nil {
			e.log.WithError(errClose).Errorf("Engine %v failed to close src engine %v client with address %v"+
				" during snapshot clone", e.Name, srcEngineName, srcEngineAddress)
		}
	}()

	srcEngine, err := srcEngineServiceCli.EngineGet(srcEngineName)
	if err != nil {
		return err
	}
	srcReplicas, err := srcEngineServiceCli.EngineReplicaList(srcEngineName)
	if err != nil {
		return err
	}

	srcReplicaCandidates := map[string]replicaCandidate{}
	for rName, mode := range srcEngine.ReplicaModeMap {
		if mode != types.ModeRW {
			continue
		}
		rAddr, ok := srcEngine.ReplicaAddressMap[rName]
		if !ok {
			continue
		}
		r, ok := srcReplicas[rName]
		if !ok {
			continue
		}
		srcReplicaCandidates[rName] = replicaCandidate{ip: r.IP, lvsUUID: r.LvsUUID, address: rAddr}
	}

	srcReplicaName := ""
	srcReplicaAddress := ""
	for rName, cand := range srcReplicaCandidates {
		if cand.ip == dstReplica.IP && cand.lvsUUID == dstReplica.LvsUUID {
			srcReplicaName = rName
			srcReplicaAddress = cand.address
			break
		}
	}

	if srcReplicaName == "" || srcReplicaAddress == "" {
		if cloneMode == spdkrpc.CloneMode_CLONE_MODE_LINKED_CLONE {
			return fmt.Errorf("cannot find the src replica at the same address %v and on same LvsUUID %v as the "+
				"dst replica", dstReplica.IP, dstReplica.LvsUUID)
		}
		for rName, cand := range srcReplicaCandidates {
			srcReplicaName = rName
			srcReplicaAddress = cand.address
			break
		}
	}

	if srcReplicaName == "" || srcReplicaAddress == "" {
		return fmt.Errorf("cannot find the src replica for cloning")
	}

	return dstReplicaServiceCli.ReplicaSnapshotCloneDstStart(dstReplicaName, snapshotName, srcReplicaName, srcReplicaAddress, cloneMode)
}

func (e *Engine) getReplicaSnapshotHashStatus(replicaName, replicaAddress, snapshotName string) (*spdkrpc.ReplicaSnapshotHashStatusResponse, error) {
	replicaServiceCli, err := GetServiceClient(replicaAddress)
	if err != nil {
		return nil, err
	}
	defer func() {
		if errClose := replicaServiceCli.Close(); errClose != nil {
			e.log.WithError(errClose).Errorf("Failed to close replica client with address %s during get hash status", replicaAddress)
		}
	}()

	return replicaServiceCli.ReplicaSnapshotHashStatus(replicaName, snapshotName)
}

func (e *Engine) ReplicaList(spdkClient *spdkclient.Client) (ret map[string]*api.Replica, err error) {
	e.Lock()
	defer e.Unlock()

	replicas := map[string]*api.Replica{}

	for name, replicaStatus := range e.ReplicaStatusMap {
		replicaServiceCli, err := GetServiceClient(replicaStatus.Address)
		if err != nil {
			e.log.WithError(err).Errorf("Failed to get service client for replica %s with address %s during list replicas", name, replicaStatus.Address)
			continue
		}

		func() {
			defer func() {
				if errClose := replicaServiceCli.Close(); errClose != nil {
					e.log.WithError(errClose).Errorf("Failed to close replica %s client with address %s during list replicas", name, replicaStatus.Address)
				}
			}()

			replica, err := replicaServiceCli.ReplicaGet(name)
			if err != nil {
				e.log.WithError(err).Errorf("Failed to get replica %s with address %s", name, replicaStatus.Address)
				return
			}

			replicas[name] = replica
		}()
	}

	return replicas, nil
}

func (e *Engine) SetErrorState() {
	needUpdate := false

	e.Lock()
	defer func() {
		e.Unlock()

		if needUpdate {
			e.UpdateCh <- nil
		}
	}()

	if e.State != types.InstanceStateStopped && e.State != types.InstanceStateError {
		e.State = types.InstanceStateError
		needUpdate = true
	}
}

func (e *Engine) BackupCreate(backupName, volumeName, engineName, snapshotName, backingImageName, backingImageChecksum string,
	labels []string, backupTarget string, credential map[string]string, concurrentLimit int32, compressionMethod, storageClassName string, size uint64) (*BackupCreateInfo, error) {
	e.log.Infof("Creating backup %s", backupName)

	e.Lock()
	defer func() {
		e.Unlock()
		e.UpdateCh <- nil
	}()

	replicaName, replicaAddress := "", ""
	for name, replicaStatus := range e.ReplicaStatusMap {
		if replicaStatus.Mode != types.ModeRW {
			continue
		}
		replicaName = name
		replicaAddress = replicaStatus.Address
		break
	}

	e.log.Infof("Creating backup %s for volume %s on replica %s address %s", backupName, volumeName, replicaName, replicaAddress)

	replicaServiceCli, err := GetServiceClient(replicaAddress)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "%v", err)
	}
	defer func() {
		if errClose := replicaServiceCli.Close(); errClose != nil {
			e.log.WithError(errClose).Errorf("Failed to close replica %s client with address %s during create backup", replicaName, replicaAddress)
		}
	}()

	recv, err := replicaServiceCli.ReplicaBackupCreate(&client.BackupCreateRequest{
		BackupName:           backupName,
		SnapshotName:         snapshotName,
		VolumeName:           volumeName,
		ReplicaName:          replicaName,
		Size:                 size,
		BackupTarget:         backupTarget,
		StorageClassName:     storageClassName,
		BackingImageName:     backingImageName,
		BackingImageChecksum: backingImageChecksum,
		CompressionMethod:    compressionMethod,
		ConcurrentLimit:      concurrentLimit,
		Labels:               labels,
		Credential:           credential,
	})
	if err != nil {
		return nil, err
	}
	return &BackupCreateInfo{
		BackupName:     recv.Backup,
		IsIncremental:  recv.IsIncremental,
		ReplicaAddress: replicaAddress,
	}, nil
}

func (e *Engine) BackupStatus(backupName, replicaAddress string) (*spdkrpc.BackupStatusResponse, error) {
	e.Lock()
	defer e.Unlock()

	found := false
	for name, replicaStatus := range e.ReplicaStatusMap {
		if replicaStatus.Address == replicaAddress {
			if replicaStatus.Mode != types.ModeRW {
				return nil, grpcstatus.Errorf(grpccodes.Internal, "replica %s is not in RW mode", name)
			}
			found = true
			break
		}
	}

	if !found {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "replica address %s is not found in engine %s for getting backup %v status", replicaAddress, e.Name, backupName)
	}

	replicaServiceCli, err := GetServiceClient(replicaAddress)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "%v", err)
	}
	defer func() {
		if errClose := replicaServiceCli.Close(); errClose != nil {
			e.log.WithError(errClose).Errorf("Failed to close replica client with address %s during get backup %s status", replicaAddress, backupName)
		}
	}()

	return replicaServiceCli.ReplicaBackupStatus(backupName)
}

func (e *Engine) BackupRestore(spdkClient *spdkclient.Client, backupUrl, engineName, snapshotName string, credential map[string]string, concurrentLimit int32) (*spdkrpc.EngineBackupRestoreResponse, error) {
	e.log.Infof("Restoring backup %s", backupUrl)

	e.Lock()
	defer e.Unlock()

	resp := &spdkrpc.EngineBackupRestoreResponse{
		Errors: map[string]string{},
	}

	backupInfo, err := backupstore.InspectBackup(backupUrl)
	if err != nil {
		for _, replicaStatus := range e.ReplicaStatusMap {
			resp.Errors[replicaStatus.Address] = err.Error()
		}
		return resp, nil
	}

	if backupInfo.VolumeSize != int64(e.SpecSize) {
		return nil, fmt.Errorf("the backup volume %v size %v must be the same as the Longhorn volume size %v", backupInfo.VolumeName, backupInfo.VolumeSize, e.SpecSize)
	}

	e.log.Infof("Deleting raid bdev %s before restoration", e.Name)
	if _, err := spdkClient.BdevRaidDelete(e.Name); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return nil, errors.Wrapf(err, "failed to delete raid bdev %s before restoration", e.Name)
	}

	e.log.Info("Disconnecting all replicas before restoration")
	for replicaName, replicaStatus := range e.ReplicaStatusMap {
		if err := disconnectNVMfBdev(spdkClient, replicaStatus.BdevName, disconnectMaxRetries, disconnectRetryInterval); err != nil {
			e.log.Infof("Failed to remove replica %s before restoration", replicaName)
			return nil, errors.Wrapf(err, "failed to remove replica %s before restoration", replicaName)
		}
		replicaStatus.BdevName = ""
	}

	e.IsRestoring = true

	switch {
	case snapshotName != "":
		e.RestoringSnapshotName = snapshotName
		e.log.Infof("Using input snapshot name %s for the restore", e.RestoringSnapshotName)
	case len(e.SnapshotMap) == 0:
		e.RestoringSnapshotName = util.UUID()
		e.log.Infof("Using new generated snapshot name %s for the full restore", e.RestoringSnapshotName)
	case e.RestoringSnapshotName != "":
		e.log.Infof("Using existing snapshot name %s for the incremental restore", e.RestoringSnapshotName)
	default:
		e.RestoringSnapshotName = util.UUID()
		e.log.Infof("Using new generated snapshot name %s for the incremental restore because e.FinalSnapshotName is empty", e.RestoringSnapshotName)
	}

	defer func() {
		go func() {
			if err := e.completeBackupRestore(spdkClient); err != nil {
				e.log.WithError(err).Warn("Failed to complete backup restore")
			}
		}()
	}()

	for replicaName, replicaStatus := range e.ReplicaStatusMap {
		e.log.Infof("Restoring backup on replica %s address %s", replicaName, replicaStatus.Address)

		replicaServiceCli, err := GetServiceClient(replicaStatus.Address)
		if err != nil {
			e.log.WithError(err).Errorf("Failed to restore backup on replica %s with address %s", replicaName, replicaStatus.Address)
			resp.Errors[replicaStatus.Address] = err.Error()
			continue
		}

		func() {
			defer func() {
				if errClose := replicaServiceCli.Close(); errClose != nil {
					e.log.WithError(errClose).Errorf("Failed to close replica %s client with address %s during restore backup", replicaName, replicaStatus.Address)
				}
			}()

			err = replicaServiceCli.ReplicaBackupRestore(&client.BackupRestoreRequest{
				BackupUrl:       backupUrl,
				ReplicaName:     replicaName,
				SnapshotName:    e.RestoringSnapshotName,
				Credential:      credential,
				ConcurrentLimit: concurrentLimit,
			})
			if err != nil {
				e.log.WithError(err).Errorf("Failed to restore backup on replica %s address %s", replicaName, replicaStatus.Address)
				resp.Errors[replicaStatus.Address] = err.Error()
			}
		}()
	}

	return resp, nil
}

func (e *Engine) completeBackupRestore(spdkClient *spdkclient.Client) error {
	if err := e.waitForRestoreComplete(); err != nil {
		return errors.Wrapf(err, "failed to wait for restore complete")
	}

	return e.BackupRestoreFinish(spdkClient)
}

func (e *Engine) waitForRestoreComplete() error {
	periodicChecker := time.NewTicker(time.Duration(restorePeriodicRefreshInterval.Seconds()) * time.Second)
	defer periodicChecker.Stop()

	var err error
	for range periodicChecker.C {
		isReplicaRestoreCompleted := true
		for replicaName, replicaStatus := range e.ReplicaStatusMap {
			if replicaStatus.Mode != types.ModeRW {
				continue
			}

			isReplicaRestoreCompleted, err = e.isReplicaRestoreCompleted(replicaName, replicaStatus.Address)
			if err != nil {
				return errors.Wrapf(err, "failed to check replica %s restore status", replicaName)
			}

			if !isReplicaRestoreCompleted {
				break
			}
		}

		if isReplicaRestoreCompleted {
			e.log.Info("Backup restoration completed successfully")
			return nil
		}
	}

	return errors.Errorf("failed to wait for engine %s restore complete", e.Name)
}

func (e *Engine) isReplicaRestoreCompleted(replicaName, replicaAddress string) (bool, error) {
	log := e.log.WithFields(logrus.Fields{
		"replica": replicaName,
		"address": replicaAddress,
	})
	log.Trace("Checking replica restore status")

	replicaServiceCli, err := GetServiceClient(replicaAddress)
	if err != nil {
		return false, errors.Wrapf(err, "failed to get replica %v service client %s", replicaName, replicaAddress)
	}
	defer func() {
		if errClose := replicaServiceCli.Close(); errClose != nil {
			log.WithError(errClose).Errorf("Failed to close replica %s client with address %s during check restore status", replicaName, replicaAddress)
		}
	}()

	status, err := replicaServiceCli.ReplicaRestoreStatus(replicaName)
	if err != nil {
		return false, errors.Wrapf(err, "failed to check replica %s restore status", replicaName)
	}

	return !status.IsRestoring, nil
}

func (e *Engine) BackupRestoreFinish(spdkClient *spdkclient.Client) error {
	updateRequired := false

	e.Lock()
	defer func() {
		e.Unlock()
		if updateRequired {
			e.UpdateCh <- nil
		}
	}()

	replicaBdevList := []string{}
	for replicaName, replicaStatus := range e.ReplicaStatusMap {
		replicaAddress := replicaStatus.Address
		replicaIP, replicaPort, err := net.SplitHostPort(replicaAddress)
		if err != nil {
			return err
		}
		e.log.Infof("Attaching replica %s with address %s before finishing restoration", replicaName, replicaAddress)
		nvmeBdevNameList, err := spdkClient.BdevNvmeAttachController(replicaName, helpertypes.GetNQN(replicaName), replicaIP, replicaPort,
			spdktypes.NvmeTransportTypeTCP, spdktypes.NvmeAddressFamilyIPv4,
			int32(e.ctrlrLossTimeout), replicaReconnectDelaySec, int32(e.fastIOFailTimeoutSec), replicaMultipath)
		if err != nil {
			return err
		}

		if len(nvmeBdevNameList) != 1 {
			return fmt.Errorf("got unexpected nvme bdev list %v", nvmeBdevNameList)
		}

		replicaStatus.BdevName = nvmeBdevNameList[0]
		replicaStatus.Mode = types.ModeRW

		replicaBdevList = append(replicaBdevList, replicaStatus.BdevName)
	}

	e.log.Infof("Creating raid bdev %s with replicas %+v before finishing restoration", e.Name, replicaBdevList)
	if _, err := spdkClient.BdevRaidCreate(e.Name, spdktypes.BdevRaidLevel1, 0, replicaBdevList, ""); err != nil {
		if !jsonrpc.IsJSONRPCRespErrorFileExists(err) {
			e.log.WithError(err).Errorf("Failed to create raid bdev before finishing restoration")
			return err
		}
	}

	e.IsRestoring = false
	e.checkAndUpdateInfoFromReplicaNoLock()
	updateRequired = true

	return nil
}

func (e *Engine) RestoreStatus() (*spdkrpc.RestoreStatusResponse, error) {
	resp := &spdkrpc.RestoreStatusResponse{
		Status: map[string]*spdkrpc.ReplicaRestoreStatusResponse{},
	}

	e.Lock()
	defer e.Unlock()

	for replicaName, replicaStatus := range e.ReplicaStatusMap {
		if replicaStatus.Mode != types.ModeRW {
			continue
		}

		restoreStatus, err := e.getReplicaRestoreStatus(replicaName, replicaStatus.Address)
		if err != nil {
			return nil, err
		}
		resp.Status[replicaStatus.Address] = restoreStatus
	}

	return resp, nil
}

func (e *Engine) getReplicaRestoreStatus(replicaName, replicaAddress string) (*spdkrpc.ReplicaRestoreStatusResponse, error) {
	replicaServiceCli, err := GetServiceClient(replicaAddress)
	if err != nil {
		return nil, err
	}
	defer func() {
		if errClose := replicaServiceCli.Close(); errClose != nil {
			e.log.WithError(errClose).Errorf("Failed to close replica client with address %s during get restore status", replicaAddress)
		}
	}()

	status, err := replicaServiceCli.ReplicaRestoreStatus(replicaName)
	if err != nil {
		return nil, err
	}

	return status, nil
}

// Expand performs an online volume expansion for the Longhorn Engine using SPDK.
// It expands the underlying replica logical volumes (lvol), recreates the SPDK RAID bdev,
// suspends and resumes frontend I/O as needed, and ensures cleanup and status updates on failure.
func (e *Engine) Expand(spdkClient *spdkclient.Client, size uint64) (err error) {
	// Add precheck
	requireExpansion, err := e.ExpandPrecheck(spdkClient, size)
	if err != nil {
		return err
	}

	e.Lock()
	originalSize := e.SpecSize
	if !requireExpansion {
		if e.SpecSize < size {
			e.SpecSize = size
		}
		// Clear stale expansion error from a previous partial failure,
		// since there is nothing left to expand.
		e.lastExpansionError = ""
		e.lastExpansionFailedAt = ""
		e.Unlock()
		return nil
	}
	defer e.Unlock()
	if e.isExpanding {
		return fmt.Errorf("%w", ErrExpansionInProgress)
	}
	e.isExpanding = true
	e.lastExpansionFailedAt = ""
	e.lastExpansionError = ""

	e.log.Info("Expanding engine frontend")

	defer func() {
		e.isExpanding = false
		e.finishExpansion(originalSize, size, err)
	}()

	var expandErr error

	replicaClients, err := e.getReplicaClients()
	if err != nil {
		return err
	}
	defer e.closeReplicaClients(replicaClients)

	e.log.Infof("Stopping to expose RAID bdev for engine %s", e.Name)
	switch e.Frontend {
	case types.FrontendUBLK:
		return fmt.Errorf("not support ublk frontend for expansion for engine %s", e.Name)
	case types.FrontendSPDKTCPBlockdev, types.FrontendSPDKTCPNvmf:
		if err := spdkClient.StopExposeBdev(e.NvmeTcpTarget.Nqn); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return errors.Wrapf(err, "failed to stop exposing bdev for engine %s", e.Name)
		}
	}

	e.log.Infof("Tearing down RAID bdev for engine %s", e.Name)
	raidBdevUUID, err := e.tearDownRaidBdev(spdkClient)
	if err != nil {
		return errors.Wrap(err, "failed to tear down expansion")
	}
	if e.RaidBdevUUID == "" {
		e.RaidBdevUUID = raidBdevUUID
	}

	// Perform expansion
	// We should always try to reconstruct the RAID bdev even if the expansion fails.
	if err := e.expandReplicas(spdkClient, replicaClients, size); err != nil {
		e.log.WithError(err).Errorf("Failed to expand replicas for engine %s", e.Name)
		// If expansion failed, we should return the error to the caller,
		// but we still need to Reconstruct the RAID bdev and Expose it
		// to make sure volume is still usable (with old size).
		// We will capture this error and return it at the end of the function.
		// We don't return here because we need to reconstruct the RAID bdev.
		// If we return here, the volume will be lost as we already tear down the RAID bdev.
		// We will return this error after the deferred functions are executed.
		// However, the err variable is named return variable, so we can just assign it.
		// But we need to be careful not to overwrite it with nil if subsequent steps succeed.
		// So we use a separate variable.
		expandErr = err
	}

	e.log.Infof("Reconstructing RAID bdev for engine %s", e.Name)
	if err := e.reconstructRaidBdev(spdkClient, e.RaidBdevUUID); err != nil {
		return errors.Wrap(err, "failed to reconstruct RAID bdev")
	}

	switch e.Frontend {
	case types.FrontendSPDKTCPBlockdev, types.FrontendSPDKTCPNvmf:
		e.log.Infof("Starting to expose RAID bdev for engine target %v on %v:%v",
			e.Name, e.NvmeTcpTarget.IP, e.NvmeTcpTarget.Port)
		if err := spdkClient.StartExposeBdev(e.NvmeTcpTarget.Nqn, e.Name, e.NvmeTcpTarget.Nguid,
			e.NvmeTcpTarget.IP, strconv.Itoa(int(e.NvmeTcpTarget.Port))); err != nil {
			return errors.Wrapf(err, "failed to start exposing RAID bdev for engine target %v", e.Name)
		}
	case types.FrontendEmpty:
		e.log.Infof("Skipping RAID bdev exposure for engine %s after expansion because frontend is empty", e.Name)
	}

	return expandErr
}

func (e *Engine) finishExpansion(fromSize, toSize uint64, err error) {
	if err != nil {
		e.SpecSize = fromSize
		e.State = types.InstanceStateError
		e.ErrorMsg = err.Error()
		e.lastExpansionError = errors.Wrap(err, "engine failed to expand expansion").Error()
		e.lastExpansionFailedAt = time.Now().UTC().Format(time.RFC3339Nano)

		e.log.WithError(err).Errorf("Engine %s failed to expand", e.Name)
		e.log.Infof("Failed to expand from size %v to %v", fromSize, toSize)
		return
	}

	e.State = types.InstanceStateRunning
	e.ErrorMsg = ""
	if e.lastExpansionError != "" {
		e.SpecSize = fromSize
		if e.lastExpansionFailedAt == "" {
			e.lastExpansionFailedAt = time.Now().UTC().Format(time.RFC3339Nano)
		}
		e.log.Warnf("Partially failed to expand from size %v to %v; keeping engine size at %v: %v",
			fromSize, toSize, fromSize, e.lastExpansionError)
		return
	}

	e.SpecSize = toSize
	e.log.Infof("Succeeded to expand from size %v to %v", fromSize, toSize)
}

func (e *Engine) tearDownRaidBdev(spdkClient *spdkclient.Client) (bdevUUID string, err error) {
	bdevRaid, err := spdkClient.BdevRaidGet(e.Name, 0)
	if err != nil {
		if jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			// RAID bdev does not exist, do nothing
			return "", nil
		}
		return "", errors.Wrapf(err, "failed to get RAID bdev %s", e.Name)
	}
	if len(bdevRaid) == 0 {
		// RAID already deleted, do nothing
		return "", nil
	}

	bdevUUID = bdevRaid[0].UUID

	deleted, err := spdkClient.BdevRaidDelete(e.Name)
	if err != nil {
		if jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			e.log.WithField("engineName", e.Name).Info("RAID bdev already deleted")
			return bdevUUID, nil
		}
		return bdevUUID, err
	}

	if deleted {
		return bdevUUID, nil
	}

	return bdevUUID, fmt.Errorf("failed to delete RAID bdev %s", e.Name)
}

func (e *Engine) expandReplicas(spdkClient *spdkclient.Client, replicaClients map[string]*client.SPDKClient, size uint64) error {
	e.log.Info("Expanding replicas")

	var (
		wg     sync.WaitGroup
		mu     sync.Mutex
		failed = make(map[string]error)
	)

	recordFailure := func(replicaName string, err error) {
		if err == nil {
			return
		}
		mu.Lock()
		failed[replicaName] = err
		mu.Unlock()
	}

	for replicaName, replicaClient := range replicaClients {
		replicaName, replicaClient := replicaName, replicaClient

		wg.Add(1)
		go func() {
			defer wg.Done()
			defer func() {
				if r := recover(); r != nil {
					recordFailure(replicaName, fmt.Errorf("panic during replica expansion: %v", r))
					e.log.WithField("replica", replicaName).Errorf("Panic during replica expansion: %v", r)
				}
			}()

			if err := e.expandSingleReplica(spdkClient, replicaName, replicaClient, size); err != nil {
				recordFailure(replicaName, err)
			}
		}()
	}

	wg.Wait()

	return e.handleReplicaExpandResult(replicaClients, failed)
}

func (e *Engine) expandSingleReplica(spdkClient *spdkclient.Client, replicaName string, replicaClient *client.SPDKClient, size uint64) error {
	replicaStatus, ok := e.ReplicaStatusMap[replicaName]
	if !ok {
		e.log.WithField("replica", replicaName).Warn("Replica not found in status map")
		return nil
	}

	replica, err := replicaClient.ReplicaGet(replicaName)
	if err != nil {
		return errors.Wrap(err, "get replica failure")
	}

	if replica.SpecSize == size {
		return nil
	}

	if err := disconnectNVMfBdev(spdkClient, replicaStatus.BdevName, disconnectMaxRetries, disconnectRetryInterval); err != nil {
		return err
	}

	if err := replicaClient.ReplicaExpand(replicaName, size); err != nil {
		return err
	}

	_, err = connectNVMfBdev(spdkClient, replicaName, replicaStatus.Address, e.ctrlrLossTimeout, e.fastIOFailTimeoutSec, maxRetries, retryInterval)
	return err
}

func (e *Engine) handleReplicaExpandResult(replicaClients map[string]*client.SPDKClient, failed map[string]error) error {
	if len(failed) == 0 {
		e.log.Info("All replicas expand success")
		return nil
	}

	aggregatedErr := aggregateReplicaErrors(failed)

	if len(failed) == len(replicaClients) {
		e.log.WithFields(logrus.Fields{"failedReplicas": aggregatedErr}).
			Error("All replicas failed to expand")
		return fmt.Errorf("all replicas failed to expand; aborting RAID recreation: %+v", aggregatedErr)
	}

	e.markReplicasERR(failed)
	e.lastExpansionError = fmt.Sprintf("%+v", aggregatedErr)
	e.log.WithFields(logrus.Fields{"failedReplicas": aggregatedErr}).
		Warn("Some replicas failed to expand and have been marked as ERR")
	return nil
}

func aggregateReplicaErrors(failed map[string]error) map[string]string {
	out := make(map[string]string, len(failed))
	for replicaName, err := range failed {
		out[replicaName] = err.Error()
	}
	return out
}

func (e *Engine) markReplicasERR(failed map[string]error) {
	for replicaName := range failed {
		if status, ok := e.ReplicaStatusMap[replicaName]; ok {
			status.Mode = types.ModeERR
		}
	}
}

func (e *Engine) reconstructRaidBdev(spdkClient *spdkclient.Client, bdevRaidUUID string) (err error) {
	e.log.WithFields(logrus.Fields{
		"engineName": e.Name,
		"volumeName": e.VolumeName,
		"frontend":   e.Frontend,
	}).Info("Reconstructing RAID bdev")

	// create the same name of raid bdev
	replicaBdevList := []string{}
	for _, replicaStatus := range e.ReplicaStatusMap {
		if replicaStatus.Mode != types.ModeRW {
			continue
		}
		if replicaStatus.BdevName == "" {
			continue
		}
		replicaBdevList = append(replicaBdevList, replicaStatus.BdevName)
	}
	if len(replicaBdevList) == 0 {
		return fmt.Errorf("no healthy replica bdevs available for RAID creation")
	}

	if _, err := spdkClient.BdevRaidCreate(e.Name, spdktypes.BdevRaidLevel1, 0, replicaBdevList, bdevRaidUUID); err != nil {
		return err
	}

	// wait the raid bdev is created
	backoff := wait.Backoff{
		Steps:    10,
		Duration: time.Second,
		Factor:   1.5,
		Jitter:   0.1,
		Cap:      time.Second * 10,
	}

	if err := retry.RetryOnConflict(backoff, func() error {
		_, err := spdkClient.BdevRaidGet(e.Name, 0)
		return err
	}); err != nil {
		return err
	}

	return nil
}

func (e *Engine) ExpandPrecheck(spdkClient *spdkclient.Client, size uint64) (requireExpansion bool, err error) {
	e.Lock()
	defer e.Unlock()

	e.log.Info("Prechecking engine expansion")

	if e.isExpanding {
		return false, fmt.Errorf("%w", ErrExpansionInProgress)
	}

	if e.IsRestoring {
		return false, fmt.Errorf("%w", ErrRestoringInProgress)
	}

	defer func() {
		if err != nil {
			e.log.WithError(err).Error("Engine precheck expansion failed")
		} else {
			e.log.Infof("Engine precheck expansion result: requireExpansion=%v", requireExpansion)
		}
	}()

	replicaClients, err := e.getReplicaClients()
	if err != nil {
		return false, errors.Wrapf(err, "failed to get replica clients")
	}
	defer e.closeReplicaClients(replicaClients)

	// Ensure all replicas are in RW mode and have the same size
	if len(e.ReplicaStatusMap) == 0 {
		return false, fmt.Errorf("cannot expand engine with no replica")
	}

	currentReplicaSize := uint64(0)
	for replicaName, replicaStatus := range e.ReplicaStatusMap {
		e.log.Infof("Checking replica %s status", replicaName)
		if replicaStatus.Mode != types.ModeRW {
			return false, fmt.Errorf("cannot expand engine with replica %s in mode %v", replicaName, replicaStatus.Mode)
		}

		replicaClient, ok := replicaClients[replicaName]
		if !ok {
			return false, fmt.Errorf("cannot find client for replica %s", replicaName)
		}
		replica, err := replicaClient.ReplicaGet(replicaName)
		if err != nil {
			return false, errors.Wrapf(err, "cannot get replica %s before expansion", replicaName)
		}

		if currentReplicaSize == 0 {
			currentReplicaSize = replica.SpecSize
			continue
		}

		if currentReplicaSize != replica.SpecSize {
			return false, fmt.Errorf("cannot expand engine with replicas in different sizes: replica %s has size %v while other replicas have size %v",
				replicaName, replica.SpecSize, currentReplicaSize)
		}
	}

	if currentReplicaSize > size {
		return false, fmt.Errorf("%w: cannot expand engine to a smaller size %v, current replica size %v",
			ErrExpansionInvalidSize, size, currentReplicaSize)
	}
	if currentReplicaSize == size {
		e.log.Infof("Replicas already at requested size %v, skipping expansion", size)
		return false, nil // no need to expand
	}

	return true, nil
}

func (e *Engine) ValidateAndUpdate(spdkClient *spdkclient.Client) (err error) {
	updateRequired := false

	e.Lock()
	defer func() {
		e.Unlock()

		if updateRequired {
			e.UpdateCh <- nil
		}
	}()

	if e.shouldSkipValidateAndUpdateNoLock() {
		return nil
	}

	bdevMap, err := GetBdevMap(spdkClient)
	if err != nil {
		return err
	}

	defer e.applyValidateAndUpdateErrorNoLock(err, &updateRequired)

	bdevRaid, err := e.getRaidBdevNoLock(bdevMap)
	if err != nil {
		return err
	}

	if err := e.validateAndMaybeAdjustSpecSizeNoLock(bdevRaid); err != nil {
		return err
	}

	containValidReplica := e.validateReplicaStatusMapNoLock(bdevMap, &updateRequired)

	e.log.UpdateLoggerWithWarnOnFailure(logrus.Fields{
		"replicaStatusMap": e.ReplicaStatusMap,
	}, "Failed to update logger with replica status map during engine creation")

	if !containValidReplica {
		e.State = types.InstanceStateError
		e.log.Error("Engine had no RW replica found at the end of ValidateAndUpdate, will be marked as error")
		updateRequired = true
		// TODO: should we delete the engine automatically here?
	}

	e.checkAndUpdateInfoFromReplicaNoLock()

	return nil
}

func (e *Engine) shouldSkipValidateAndUpdateNoLock() bool {
	if e.IsRestoring {
		e.log.Debug("Engine is restoring, will skip the validation and update")
		return true
	}

	if e.isExpanding {
		e.log.Debug("Engine is expanding, will skip the validation and update")
		return true
	}

	// Syncing with the SPDK TGT server only when the engine is running.
	if e.State != types.InstanceStateRunning {
		return true
	}

	return false
}

func (e *Engine) applyValidateAndUpdateErrorNoLock(err error, updateRequired *bool) {
	// TODO: we may not need to mark the engine as ERR for each error
	if err != nil {
		if e.State != types.InstanceStateError {
			e.State = types.InstanceStateError
			e.log.WithError(err).Error("Found error during engine validation and update")
			*updateRequired = true
		}
		e.ErrorMsg = err.Error()
		return
	}

	if e.State != types.InstanceStateError {
		e.ErrorMsg = ""
	}
}

func (e *Engine) getRaidBdevNoLock(bdevMap map[string]*spdktypes.BdevInfo) (*spdktypes.BdevInfo, error) {
	bdevRaid := bdevMap[e.Name]
	if spdktypes.GetBdevType(bdevRaid) != spdktypes.BdevTypeRaid {
		return nil, fmt.Errorf("cannot find a raid bdev for engine %v", e.Name)
	}
	return bdevRaid, nil
}

func (e *Engine) validateAndMaybeAdjustSpecSizeNoLock(bdevRaid *spdktypes.BdevInfo) error {
	bdevRaidSize := bdevRaid.NumBlocks * uint64(bdevRaid.BlockSize)

	if e.SpecSize > bdevRaidSize {
		// not directly return error
		//
		// If the volume is not attached and do the expand
		// At first, we create and attach the engine with new size, but not yet to expand
		// it will cause infinite loop for size mismatching
		// loop to destroy and create engine
		// and there is no chance to execute EngineExpand()
		//
		// wait the lh-manager to reconcile engine CR and call EngineExpand()

		e.SpecSize = bdevRaidSize
		e.log.Warnf("found mismatching between engine spec size %d and actual raid bdev size %d for engine %s", e.SpecSize, bdevRaidSize, e.Name)
		return nil
	}

	if e.SpecSize < bdevRaidSize {
		// should not happen
		return fmt.Errorf("engine spec size %d is smaller than actual raid bdev size %d for engine %s", e.SpecSize, bdevRaidSize, e.Name)
	}

	return nil
}

func (e *Engine) validateReplicaStatusMapNoLock(bdevMap map[string]*spdktypes.BdevInfo, updateRequired *bool) bool {
	containValidReplica := false

	for replicaName, replicaStatus := range e.ReplicaStatusMap {
		if replicaStatus.Address == "" || replicaStatus.BdevName == "" {
			if replicaStatus.Mode != types.ModeERR {
				e.log.Errorf("Engine marked replica %s mode from %v to ERR since its address %s or bdev name %s is empty during ValidateAndUpdate", replicaName, replicaStatus.Mode, replicaStatus.Address, replicaStatus.BdevName)
				replicaStatus.Mode = types.ModeERR
				*updateRequired = true
			}
		}

		if replicaStatus.Mode != types.ModeRW && replicaStatus.Mode != types.ModeWO && replicaStatus.Mode != types.ModeERR {
			e.log.Errorf("Engine found replica %s invalid mode %v during ValidateAndUpdate", replicaName, replicaStatus.Mode)
			replicaStatus.Mode = types.ModeERR
			*updateRequired = true
		}

		if replicaStatus.Mode != types.ModeERR {
			e.log.Debugf("Engine validating replica %s with bdev name %s and address %s during ValidateAndUpdate", replicaName, replicaStatus.BdevName, replicaStatus.Address)
			mode, err := e.validateAndUpdateReplicaNvme(replicaName, bdevMap[replicaStatus.BdevName])
			if err != nil {
				e.log.WithError(err).Errorf("Engine found valid NVMe for replica %v, will update the mode from %s to ERR during ValidateAndUpdate", replicaName, replicaStatus.Mode)
				replicaStatus.Mode = types.ModeERR
				*updateRequired = true
			} else if replicaStatus.Mode != mode {
				replicaStatus.Mode = mode
				*updateRequired = true
			}
		}

		if replicaStatus.Mode == types.ModeRW {
			containValidReplica = true
		}
	}

	return containValidReplica
}

type replicaInspection struct {
	replica           *api.Replica
	ancestor          *api.Lvol
	foundBackingImage bool
	foundSnapshot     bool
}

// checkAndUpdateInfoFromReplicaNoLock refreshes engine-level info (SnapshotMap, Head,
// ActualSize) from the replicas. It iterates ReplicaStatusMap, inspects each RW/WO
// replica, resolves its ancestor lineage, and selects the replica with the earliest
// ancestor CreationTime as the info source. Must be called with the Engine lock held.
func (e *Engine) checkAndUpdateInfoFromReplicaNoLock() {
	replicaMap := map[string]*api.Replica{}
	replicaAncestorMap := map[string]*api.Lvol{}
	hasBackingImage := false
	hasSnapshot := false

	for replicaName, replicaStatus := range e.ReplicaStatusMap {
		if !e.ensureReplicaModeForInfoUpdate(replicaName, replicaStatus) {
			continue
		}

		inspection, ok := e.inspectReplicaForInfoUpdate(replicaName, replicaStatus, hasBackingImage, hasSnapshot)
		if !ok {
			continue
		}

		if inspection.foundBackingImage {
			hasBackingImage = true
		}
		if inspection.foundSnapshot {
			hasSnapshot = true
		}

		replicaMap[replicaName] = inspection.replica
		replicaAncestorMap[replicaName] = inspection.ancestor
	}

	e.selectAndApplyEarliestReplicaInfo(replicaMap, replicaAncestorMap, hasBackingImage, hasSnapshot)
}

// ensureReplicaModeForInfoUpdate checks whether a replica's mode qualifies it
// for info update inspection. Returns true for RW and WO replicas. For any
// unexpected mode (not RW, WO, or ERR), it downgrades the mode to ERR and
// returns false.
func (e *Engine) ensureReplicaModeForInfoUpdate(replicaName string, replicaStatus *EngineReplicaStatus) bool {
	if replicaStatus.Mode == types.ModeRW || replicaStatus.Mode == types.ModeWO {
		return true
	}
	if replicaStatus.Mode != types.ModeERR {
		e.log.Warnf("Engine found unexpected mode for replica %s with address %s during info update from replica, mark the mode from %v to ERR and continue info update for other replicas",
			replicaName, replicaStatus.Address, replicaStatus.Mode)
		replicaStatus.Mode = types.ModeERR
	}
	return false
}

// inspectReplicaForInfoUpdate validates and inspects a replica as an info source candidate.
//
// Here, "info source" means the replica selected as the source of truth for this update round,
// i.e. the replica whose data may be used to update engine state such as SnapshotMap, Head,
// and ActualSize.
//
// Flow:
//  1. Build replica service client and fetch replica object.
//  2. If the replica is WO (rebuilding), only check shallow-copy state; do not use it as an info source.
//  3. If the replica is RW, resolve its ancestor (backing image snapshot / oldest snapshot / head)
//     based on current global context (hasBackingImage, hasSnapshot).
func (e *Engine) inspectReplicaForInfoUpdate(replicaName string, replicaStatus *EngineReplicaStatus, hasBackingImage bool, hasSnapshot bool) (*replicaInspection, bool) {
	replicaServiceCli, err := GetServiceClient(replicaStatus.Address)
	if err != nil {
		e.log.WithError(err).Errorf("Engine failed to get service client for replica %s with address %s, will skip this replica and continue info update for other replicas", replicaName, replicaStatus.Address)
		return nil, false
	}
	defer func() {
		if errClose := replicaServiceCli.Close(); errClose != nil {
			e.log.WithError(errClose).Errorf("Engine failed to close replica %s client with address %s during check and update info from replica", replicaName, replicaStatus.Address)
		}
	}()

	replica, err := replicaServiceCli.ReplicaGet(replicaName)
	if err != nil {
		e.log.WithError(err).Warnf("Engine failed to get replica %s with address %s, mark the mode from %v to ERR", replicaName, replicaStatus.Address, replicaStatus.Mode)
		replicaStatus.Mode = types.ModeERR
		return nil, false
	}

	if replicaStatus.Mode == types.ModeWO {
		if err := e.handleWOReplicaDuringInfoUpdate(replicaServiceCli, replicaName, replicaStatus); err != nil {
			e.log.WithError(err).Warn("Skip WO replica during info update")
		}
		return nil, false
	}

	inspection := &replicaInspection{replica: replica}
	ancestor, foundBackingImage, foundSnapshot, ok := e.resolveReplicaAncestor(replicaServiceCli, replicaName, replica, replicaStatus, hasBackingImage, hasSnapshot)
	if !ok {
		return nil, false
	}
	inspection.ancestor = ancestor
	inspection.foundBackingImage = foundBackingImage
	inspection.foundSnapshot = foundSnapshot

	return inspection, true
}

func (e *Engine) handleWOReplicaDuringInfoUpdate(replicaServiceCli *client.SPDKClient, replicaName string, replicaStatus *EngineReplicaStatus) error {
	shallowCopyStatus, err := replicaServiceCli.ReplicaRebuildingDstShallowCopyCheck(replicaName)
	if err != nil {
		return errors.Wrapf(err, "Engine failed to get rebuilding replica %s shallow copy info, will skip this replica and continue info update for other replicas", replicaName)
	}
	if shallowCopyStatus.TotalState == helpertypes.ShallowCopyStateError || shallowCopyStatus.Error != "" {
		replicaStatus.Mode = types.ModeERR
		return fmt.Errorf("Engine found rebuilding replica %s error %v during info update from replica, will mark the mode from WO to ERR and continue info update for other replicas", replicaName, shallowCopyStatus.Error)
	}
	// rebuilding replica is not used as info source
	return nil
}

// resolveReplicaAncestor determines the ancestor lvol used to compare replica lineage
// during engine info refresh.
//
// Selection order per replica:
// 1. Backing image snapshot (if the replica has a backing image)
// 2. Oldest snapshot (snapshot with empty Parent)
// 3. Head (when no snapshots exist)
//
// It also enforces cross-replica consistency for this round:
// - If any replica has backing image lineage, replicas without backing image lineage are skipped.
// - If any replica has snapshot lineage (and no backing image lineage), replicas without snapshots are skipped.
//
// Returns:
// - ancestor: selected lvol for lineage/creation-time comparison.
// - foundBackingImage: true if this replica contributes backing-image lineage.
// - foundSnapshot: true if this replica contributes snapshot lineage.
// - ok: false when the replica should be skipped (inconsistent lineage, missing ancestor, or lookup failure).
func (e *Engine) resolveReplicaAncestor(replicaServiceCli *client.SPDKClient, replicaName string,
	replica *api.Replica, replicaStatus *EngineReplicaStatus, hasBackingImage bool, hasSnapshot bool) (ancestor *api.Lvol, foundBackingImage bool, foundSnapshot bool, ok bool) {
	if replica.BackingImageName != "" {
		backingImage, err := replicaServiceCli.BackingImageGet(replica.BackingImageName, replica.LvsUUID)
		if err != nil {
			e.log.WithError(err).Warnf("Failed to get backing image %s with disk UUID %s from replica %s head parent %s, will mark the mode from %v to ERR and continue info update for other replicas", replica.BackingImageName, replica.LvsUUID, replicaName, replica.Head.Parent, replicaStatus.Mode)
			replicaStatus.Mode = types.ModeERR
			return nil, false, false, false
		}
		return backingImage.Snapshot, true, len(replica.Snapshots) > 0, true
	}

	if len(replica.Snapshots) > 0 {
		if hasBackingImage {
			e.log.Warnf("Engine found replica %s does not have a backing image while other replicas have during info update for other replicas", replicaName)
			return nil, false, false, false
		}
		for _, snapApiLvol := range replica.Snapshots {
			if snapApiLvol.Parent == "" {
				return snapApiLvol, false, true, true
			}
		}
		e.log.Warnf("Engine cannot find replica %s ancestor, will skip this replica and continue info update for other replicas", replicaName)
		return nil, false, false, false
	}

	if hasSnapshot {
		e.log.Warnf("Engine found replica %s does not have a snapshot while other replicas have during info update for other replicas", replicaName)
		return nil, false, false, false
	}
	return replica.Head, false, false, true
}

// selectAndApplyEarliestReplicaInfo chooses one replica as the engine info source
// and applies its state to the engine.
//
// From replicas that already passed inspection, it filters candidates by lineage type:
// - backing-image lineage if hasBackingImage is true
// - snapshot lineage if hasBackingImage is false and hasSnapshot is true
// - head lineage otherwise
//
// It then selects the candidate whose chosen ancestor has the earliest CreationTime.
// Once selected, it updates engine state from that replica:
// - e.SnapshotMap
// - e.Head
// - e.ActualSize
//
// If candidate switching happens and ancestor names differ, it emits a warning log.
//
// Notes:
// - Replicas with invalid/unparsable ancestor CreationTime are skipped.
// - If no valid candidate remains, engine state is left unchanged.
func (e *Engine) selectAndApplyEarliestReplicaInfo(replicaMap map[string]*api.Replica, replicaAncestorMap map[string]*api.Lvol, hasBackingImage bool, hasSnapshot bool) {
	candidateReplicaName := ""
	earliestCreationTime := time.Now()

	for replicaName, ancestorApiLvol := range replicaAncestorMap {
		if !shouldConsiderAncestor(ancestorApiLvol, replicaName, hasBackingImage, hasSnapshot) {
			continue
		}

		creationTime, err := time.Parse(time.RFC3339, ancestorApiLvol.CreationTime)
		if err != nil {
			e.log.WithError(err).Warnf("Failed to parse replica %s ancestor creation time, will skip this replica and continue info update for other replicas: %+v", replicaName, ancestorApiLvol)
			continue
		}
		if !earliestCreationTime.After(creationTime) {
			continue
		}

		earliestCreationTime = creationTime
		e.SnapshotMap = replicaMap[replicaName].Snapshots
		e.Head = replicaMap[replicaName].Head
		e.ActualSize = replicaMap[replicaName].ActualSize

		if candidateReplicaName != "" && candidateReplicaName != replicaName {
			e.logReplicaAncestorSwitch(candidateReplicaName, replicaName, replicaAncestorMap)
		}
		candidateReplicaName = replicaName
	}
}

func shouldConsiderAncestor(ancestor *api.Lvol, replicaName string, hasBackingImage bool, hasSnapshot bool) bool {
	if hasBackingImage {
		return ancestor.Name != types.VolumeHead && !IsReplicaSnapshotLvol(replicaName, ancestor.Name)
	}
	if hasSnapshot {
		return ancestor.Name != types.VolumeHead
	}
	return ancestor.Name == types.VolumeHead
}

func (e *Engine) logReplicaAncestorSwitch(prevReplica string, currReplica string, replicaAncestorMap map[string]*api.Lvol) {
	prevName := replicaAncestorMap[prevReplica].Name
	currName := replicaAncestorMap[currReplica].Name

	prevDisplay := normalizeAncestorNameForLog(e, prevName)
	currDisplay := normalizeAncestorNameForLog(e, currName)

	if prevDisplay != currDisplay {
		e.log.Warnf("Comparing with replica %s ancestor %s, replica %s has a different and earlier ancestor %s, will update info from this replica",
			prevReplica, prevName, currReplica, currName)
	}
}

func normalizeAncestorNameForLog(e *Engine, name string) string {
	if !types.IsBackingImageSnapLvolName(name) {
		return name
	}
	backingImageName, _, err := ExtractBackingImageAndDiskUUID(name)
	if err != nil {
		e.log.WithError(err).Warnf("BUG: ancestor name %v is from backingImage.Snapshot lvol name, it should be a valid backing image lvol name", name)
		return name
	}
	return backingImageName
}

func (e *Engine) validateAndUpdateReplicaNvme(replicaName string, bdev *spdktypes.BdevInfo) (types.Mode, error) {
	if bdev == nil {
		return types.ModeERR, fmt.Errorf("cannot find a bdev for replica %s", replicaName)
	}

	if err := validateReplicaBdevSize(e, replicaName, bdev); err != nil {
		return types.ModeERR, err
	}

	nvmeInfo, err := validateAndGetSingleNvmeInfo(replicaName, bdev)
	if err != nil {
		return types.ModeERR, err
	}
	if err := validateNvmeTransport(replicaName, bdev.Name, nvmeInfo); err != nil {
		return types.ModeERR, err
	}

	replicaStatus := e.ReplicaStatusMap[replicaName]
	if err := validateReplicaAddress(replicaName, bdev.Name, replicaStatus.Address, nvmeInfo); err != nil {
		return types.ModeERR, err
	}
	if err := validateControllerName(replicaName, bdev.Name, replicaStatus.BdevName); err != nil {
		return types.ModeERR, err
	}

	return replicaStatus.Mode, nil
}

func validateReplicaBdevSize(e *Engine, replicaName string, bdev *spdktypes.BdevInfo) error {
	bdevSpecSize := bdev.NumBlocks * uint64(bdev.BlockSize)
	if e.SpecSize != bdevSpecSize {
		return fmt.Errorf(
			"found mismatching between replica bdev %s spec size %d and the engine %s spec size %d during replica %s mode validation",
			bdev.Name, bdevSpecSize, e.Name, e.SpecSize, replicaName,
		)
	}
	return nil
}

func validateAndGetSingleNvmeInfo(replicaName string, bdev *spdktypes.BdevInfo) (spdktypes.NvmeNamespaceInfo, error) {
	if spdktypes.GetBdevType(bdev) != spdktypes.BdevTypeNvme {
		return spdktypes.NvmeNamespaceInfo{}, fmt.Errorf(
			"found bdev type %v rather than %v during replica %s mode validation",
			spdktypes.GetBdevType(bdev), spdktypes.BdevTypeNvme, replicaName,
		)
	}
	if bdev.DriverSpecific.Nvme == nil || len(*bdev.DriverSpecific.Nvme) != 1 {
		return spdktypes.NvmeNamespaceInfo{}, fmt.Errorf(
			"found zero or multiple NVMe info in a NVMe base bdev %v during replica %s mode validation",
			bdev.Name, replicaName,
		)
	}
	return (*bdev.DriverSpecific.Nvme)[0], nil
}

func validateNvmeTransport(replicaName, bdevName string, nvmeInfo spdktypes.NvmeNamespaceInfo) error {
	if !strings.EqualFold(string(nvmeInfo.Trid.Adrfam), string(spdktypes.NvmeAddressFamilyIPv4)) ||
		!strings.EqualFold(string(nvmeInfo.Trid.Trtype), string(spdktypes.NvmeTransportTypeTCP)) {
		return fmt.Errorf(
			"found invalid address family %s and transport type %s in a remote NVMe base bdev %s during replica %s mode validation",
			nvmeInfo.Trid.Adrfam, nvmeInfo.Trid.Trtype, bdevName, replicaName,
		)
	}
	return nil
}

func validateReplicaAddress(replicaName, bdevName, expectedAddr string, nvmeInfo spdktypes.NvmeNamespaceInfo) error {
	actualAddr := net.JoinHostPort(nvmeInfo.Trid.Traddr, nvmeInfo.Trid.Trsvcid)
	if expectedAddr != actualAddr {
		return fmt.Errorf(
			"found mismatching between replica bdev %s address %s and the NVMe bdev actual address %s during replica %s mode validation",
			bdevName, expectedAddr, actualAddr, replicaName,
		)
	}
	return nil
}

func validateControllerName(replicaName, bdevName, namespaceBdevName string) error {
	controllerName := helperutil.GetNvmeControllerNameFromNamespaceName(namespaceBdevName)
	if controllerName != replicaName {
		return fmt.Errorf(
			"found unexpected the NVMe bdev controller name %s (bdev name %s) during replica %s mode validation",
			controllerName, bdevName, replicaName,
		)
	}
	return nil
}

// SetReplicaAdder replaces the ReplicaAdder used by ReplicaAdd.
// For testing, pass a *MockReplicaAdder. Pass nil to restore production behavior.
func (e *Engine) SetReplicaAdder(adder ReplicaAdder) {
	e.Lock()
	defer e.Unlock()
	if adder == nil {
		e.replicaAdder = &realReplicaAdder{e: e}
	} else {
		// If substituting a MockReplicaAdder, inject the real adder for fallback.
		if m, ok := adder.(*MockReplicaAdder); ok && m.Real == nil {
			m.Real = &realReplicaAdder{e: e}
		}
		e.replicaAdder = adder
	}
}

// SetReplicaAddFinishUnlockedHook injects (or clears) the regression-guard
// hook for the 3-phase lock pattern in replicaAddFinish. See the field comment
// on replicaAddFinishUnlockedHook for details. Pass nil to clear.
func (e *Engine) SetReplicaAddFinishUnlockedHook(hook func()) {
	e.Lock()
	defer e.Unlock()
	e.replicaAddFinishUnlockedHook = hook
}
</file>

<file path="pkg/spdk/enginefrontend_create_test.go">
package spdk

import (
	"fmt"
	"strings"
	"sync"
	"sync/atomic"
	"time"

	lhtypes "github.com/longhorn/longhorn-spdk-engine/pkg/types"

	. "gopkg.in/check.v1"
)

func (s *TestSuite) TestEngineFrontendCreateReturnsErrorForFrontendFailure(c *C) {
	fmt.Println("Testing EngineFrontend.Create returns an error while preserving the error state")

	ef := NewEngineFrontend("ef-test", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1))
	ef.NvmeTcpFrontend = nil

	resp, err := ef.Create(nil, "10.0.0.1:9502")
	c.Assert(err, NotNil)
	c.Assert(resp, IsNil)

	got := ef.Get()
	c.Assert(got.State, Equals, string(lhtypes.InstanceStateError))
	c.Assert(got.ErrorMsg, Matches, ".*invalid NvmeTcpFrontend.*")
}

// TestCreateDoesNotHoldLockWhileSendingUpdate verifies that Create releases
// the struct lock before blocking on UpdateCh, so concurrent Get() calls
// are not starved.
func (s *TestSuite) TestCreateDoesNotHoldLockWhileSendingUpdate(c *C) {
	fmt.Println("Testing Create does not hold the lock while sending update")

	// Unbuffered channel: Create will block on the send after setting state.
	ef := NewEngineFrontend("ef-create-lock", "engine-a", "vol-a", lhtypes.FrontendEmpty, 1024, 0, 0, make(chan interface{}))

	errCh := make(chan error, 1)
	go func() {
		_, err := ef.Create(nil, "127.0.0.1:9500")
		errCh <- err
	}()

	// Wait for Create to reach Running (state is visible after Unlock,
	// before the blocking UpdateCh send).
	deadline := time.Now().Add(2 * time.Second)
	for ef.Get().State != string(lhtypes.InstanceStateRunning) {
		if time.Now().After(deadline) {
			c.Fatal("timeout waiting for engine frontend to enter running state")
		}
		time.Sleep(10 * time.Millisecond)
	}

	// Get() must not block — proves the lock was released before the
	// channel send.
	getDone := make(chan struct{}, 1)
	go func() {
		_ = ef.Get()
		getDone <- struct{}{}
	}()

	select {
	case <-getDone:
		// expected
	case <-time.After(1 * time.Second):
		c.Fatal("Get() blocked while Create is waiting on UpdateCh; lock may still be held")
	}

	// Unblock Create by draining the update channel.
	select {
	case <-ef.UpdateCh:
	case <-time.After(1 * time.Second):
		c.Fatal("timeout waiting for Create() update signal")
	}

	select {
	case err := <-errCh:
		c.Assert(err, IsNil)
	case <-time.After(1 * time.Second):
		c.Fatal("timeout waiting for Create() to return")
	}
}

// TestConcurrentCreateHasSingleWinner verifies that only one concurrent
// Create call on the same EngineFrontend succeeds; all others receive a
// precondition error.
func (s *TestSuite) TestConcurrentCreateHasSingleWinner(c *C) {
	fmt.Println("Testing concurrent Create has a single winner")

	ef := NewEngineFrontend("ef-create-concurrent", "engine-a", "vol-a", lhtypes.FrontendEmpty, 1024, 0, 0, make(chan interface{}, 32))

	const workers = 20
	startCh := make(chan struct{})
	var wg sync.WaitGroup
	var successCount int32
	errCh := make(chan error, workers)

	for i := 0; i < workers; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			<-startCh
			_, err := ef.Create(nil, "127.0.0.1:9500")
			if err == nil {
				atomic.AddInt32(&successCount, 1)
				return
			}
			errCh <- err
		}()
	}

	close(startCh)
	wg.Wait()
	close(errCh)

	c.Assert(successCount, Equals, int32(1))
	c.Assert(len(errCh), Equals, workers-1)

	for err := range errCh {
		c.Assert(err, NotNil)
		errStr := err.Error()
		c.Assert(strings.Contains(errStr, "already creating") || strings.Contains(errStr, "invalid state"), Equals, true,
			Commentf("unexpected concurrent create error: %v", err))
	}

	c.Assert(ef.Get().State, Equals, string(lhtypes.InstanceStateRunning))
}
</file>

<file path="pkg/spdk/enginefrontend_persist_test.go">
package spdk

import (
	"encoding/json"
	"os"
	"path/filepath"
	"time"

	helpertypes "github.com/longhorn/go-spdk-helper/pkg/types"

	lhtypes "github.com/longhorn/longhorn-spdk-engine/pkg/types"

	. "gopkg.in/check.v1"
)

// --- saveEngineFrontendRecord / loadEngineFrontendRecords round-trip ---

func (s *TestSuite) TestSaveAndLoadEngineFrontendRecord(c *C) {
	tmpDir := c.MkDir()

	ef := NewEngineFrontend("ef-1", "engine-a", "vol-a",
		lhtypes.FrontendSPDKTCPBlockdev, 1048576, 0, 0, make(chan interface{}, 1))
	ef.NvmeTcpFrontend.TargetIP = "10.0.0.1"
	ef.NvmeTcpFrontend.TargetPort = 3000

	err := saveEngineFrontendRecord(tmpDir, ef)
	c.Assert(err, IsNil)

	records, err := loadEngineFrontendRecords(tmpDir)
	c.Assert(err, IsNil)
	c.Assert(len(records), Equals, 1)
	c.Assert(records[0].Name, Equals, "ef-1")
	c.Assert(records[0].EngineName, Equals, "engine-a")
	c.Assert(records[0].VolumeName, Equals, "vol-a")
	c.Assert(records[0].Frontend, Equals, lhtypes.FrontendSPDKTCPBlockdev)
	c.Assert(records[0].SpecSize, Equals, uint64(1048576))
	c.Assert(records[0].TargetIP, Equals, "10.0.0.1")
	c.Assert(records[0].TargetPort, Equals, int32(3000))
}

func (s *TestSuite) TestSaveRecordPersistsTargetIPAndPort(c *C) {
	tmpDir := c.MkDir()

	ef := NewEngineFrontend("ef-nvmf", "engine-b", "vol-b",
		lhtypes.FrontendSPDKTCPNvmf, 2097152, 0, 0, make(chan interface{}, 1))
	ef.NvmeTcpFrontend.TargetIP = "192.168.1.10"
	ef.NvmeTcpFrontend.TargetPort = 4420

	err := saveEngineFrontendRecord(tmpDir, ef)
	c.Assert(err, IsNil)

	// Read raw JSON to verify fields are present.
	data, err := os.ReadFile(engineFrontendRecordPath(tmpDir, "vol-b"))
	c.Assert(err, IsNil)

	var raw map[string]interface{}
	c.Assert(json.Unmarshal(data, &raw), IsNil)
	c.Assert(raw["targetIP"], Equals, "192.168.1.10")
	c.Assert(raw["targetPort"], Equals, float64(4420)) // JSON numbers are float64
}

// --- UBLK frontend should NOT be persisted (Issue #4) ---

func (s *TestSuite) TestSaveRecordSkipsUblkFrontend(c *C) {
	tmpDir := c.MkDir()

	ef := NewEngineFrontend("ef-ublk", "engine-c", "vol-c",
		lhtypes.FrontendUBLK, 1048576, 0, 0, make(chan interface{}, 1))

	err := saveEngineFrontendRecord(tmpDir, ef)
	c.Assert(err, IsNil)

	// Verify no file was written.
	records, err := loadEngineFrontendRecords(tmpDir)
	c.Assert(err, IsNil)
	c.Assert(len(records), Equals, 0)
}

// --- Empty metadataDir is a no-op ---

func (s *TestSuite) TestSaveRecordEmptyMetadataDirIsNoop(c *C) {
	ef := NewEngineFrontend("ef-x", "engine-x", "vol-x",
		lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1))

	c.Assert(saveEngineFrontendRecord("", ef), IsNil)
}

func (s *TestSuite) TestLoadRecordsEmptyMetadataDirReturnsNil(c *C) {
	records, err := loadEngineFrontendRecords("")
	c.Assert(err, IsNil)
	c.Assert(records, IsNil)
}

func (s *TestSuite) TestLoadRecordsNonExistentDirReturnsNil(c *C) {
	records, err := loadEngineFrontendRecords("/tmp/nonexistent-dir-for-test-" + time.Now().Format("20060102150405"))
	c.Assert(err, IsNil)
	c.Assert(records, IsNil)
}

// --- Corrupted records are cleaned up (Issue #5) ---

func (s *TestSuite) TestLoadRecordsRemovesCorruptedJSON(c *C) {
	tmpDir := c.MkDir()

	// Create a corrupted record.
	volDir := filepath.Join(tmpDir, engineFrontendSubDir, "vol-corrupt")
	c.Assert(os.MkdirAll(volDir, 0700), IsNil)
	c.Assert(os.WriteFile(filepath.Join(volDir, engineFrontendRecFile), []byte("{invalid json"), 0600), IsNil)

	records, err := loadEngineFrontendRecords(tmpDir)
	c.Assert(err, IsNil)
	c.Assert(len(records), Equals, 0)

	// Verify the corrupted directory was removed.
	_, statErr := os.Stat(volDir)
	c.Assert(os.IsNotExist(statErr), Equals, true)
}

func (s *TestSuite) TestLoadRecordsRemovesRecordWithEmptyName(c *C) {
	tmpDir := c.MkDir()

	// Create a record with empty Name field.
	volDir := filepath.Join(tmpDir, engineFrontendSubDir, "vol-empty-name")
	c.Assert(os.MkdirAll(volDir, 0700), IsNil)

	record := &EngineFrontendRecord{
		Name:       "",
		VolumeName: "vol-empty-name",
		Frontend:   lhtypes.FrontendSPDKTCPBlockdev,
	}
	data, _ := json.Marshal(record)
	c.Assert(os.WriteFile(filepath.Join(volDir, engineFrontendRecFile), data, 0600), IsNil)

	records, err := loadEngineFrontendRecords(tmpDir)
	c.Assert(err, IsNil)
	c.Assert(len(records), Equals, 0)

	// Verify directory was removed.
	_, statErr := os.Stat(volDir)
	c.Assert(os.IsNotExist(statErr), Equals, true)
}

func (s *TestSuite) TestLoadRecordsRemovesRecordWithEmptyVolumeName(c *C) {
	tmpDir := c.MkDir()

	volDir := filepath.Join(tmpDir, engineFrontendSubDir, "vol-empty-volname")
	c.Assert(os.MkdirAll(volDir, 0700), IsNil)

	record := &EngineFrontendRecord{
		Name:       "ef-x",
		VolumeName: "",
		Frontend:   lhtypes.FrontendSPDKTCPBlockdev,
	}
	data, _ := json.Marshal(record)
	c.Assert(os.WriteFile(filepath.Join(volDir, engineFrontendRecFile), data, 0600), IsNil)

	records, err := loadEngineFrontendRecords(tmpDir)
	c.Assert(err, IsNil)
	c.Assert(len(records), Equals, 0)

	_, statErr := os.Stat(volDir)
	c.Assert(os.IsNotExist(statErr), Equals, true)
}

// --- Mixed valid and corrupted records ---

func (s *TestSuite) TestLoadRecordsMixedValidAndCorrupted(c *C) {
	tmpDir := c.MkDir()

	// Create a valid record.
	efValid := NewEngineFrontend("ef-valid", "engine-v", "vol-valid",
		lhtypes.FrontendSPDKTCPBlockdev, 1048576, 0, 0, make(chan interface{}, 1))
	efValid.NvmeTcpFrontend.TargetIP = "10.0.0.5"
	efValid.NvmeTcpFrontend.TargetPort = 5000
	c.Assert(saveEngineFrontendRecord(tmpDir, efValid), IsNil)

	// Create a corrupted record alongside.
	corruptDir := filepath.Join(tmpDir, engineFrontendSubDir, "vol-corrupt")
	c.Assert(os.MkdirAll(corruptDir, 0700), IsNil)
	c.Assert(os.WriteFile(filepath.Join(corruptDir, engineFrontendRecFile), []byte("not-json"), 0600), IsNil)

	records, err := loadEngineFrontendRecords(tmpDir)
	c.Assert(err, IsNil)
	c.Assert(len(records), Equals, 1)
	c.Assert(records[0].Name, Equals, "ef-valid")
	c.Assert(records[0].TargetIP, Equals, "10.0.0.5")
	c.Assert(records[0].TargetPort, Equals, int32(5000))

	// Corrupted record should be cleaned up.
	_, statErr := os.Stat(corruptDir)
	c.Assert(os.IsNotExist(statErr), Equals, true)
}

// --- removeEngineFrontendRecord ---

func (s *TestSuite) TestRemoveEngineFrontendRecord(c *C) {
	tmpDir := c.MkDir()

	ef := NewEngineFrontend("ef-rm", "engine-rm", "vol-rm",
		lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1))
	c.Assert(saveEngineFrontendRecord(tmpDir, ef), IsNil)

	// Verify exists.
	records, err := loadEngineFrontendRecords(tmpDir)
	c.Assert(err, IsNil)
	c.Assert(len(records), Equals, 1)

	// Remove.
	c.Assert(removeEngineFrontendRecord(tmpDir, "vol-rm"), IsNil)

	records, err = loadEngineFrontendRecords(tmpDir)
	c.Assert(err, IsNil)
	c.Assert(len(records), Equals, 0)
}

// --- Backward compatibility: old records without targetIP/targetPort ---

func (s *TestSuite) TestLoadRecordsBackwardCompatibleWithOldFormat(c *C) {
	tmpDir := c.MkDir()

	// Simulate an old-format record (no targetIP/targetPort fields).
	volDir := filepath.Join(tmpDir, engineFrontendSubDir, "vol-old")
	c.Assert(os.MkdirAll(volDir, 0700), IsNil)

	oldRecord := `{
  "name": "ef-old",
  "engineName": "engine-old",
  "volumeName": "vol-old",
  "frontend": "spdk-tcp-blockdev",
  "specSize": 1048576,
  "engineIP": "10.0.0.99"
}`
	c.Assert(os.WriteFile(filepath.Join(volDir, engineFrontendRecFile), []byte(oldRecord), 0600), IsNil)

	records, err := loadEngineFrontendRecords(tmpDir)
	c.Assert(err, IsNil)
	c.Assert(len(records), Equals, 1)
	c.Assert(records[0].Name, Equals, "ef-old")
	c.Assert(records[0].TargetIP, Equals, "")         // Not present in old format.
	c.Assert(records[0].TargetPort, Equals, int32(0)) // Not present in old format.
}

// --- Overwrite: saving again updates the record ---

func (s *TestSuite) TestSaveRecordOverwritesPrevious(c *C) {
	tmpDir := c.MkDir()

	ef := NewEngineFrontend("ef-ow", "engine-ow", "vol-ow",
		lhtypes.FrontendSPDKTCPNvmf, 1048576, 0, 0, make(chan interface{}, 1))
	ef.NvmeTcpFrontend.TargetIP = "10.0.0.1"
	ef.NvmeTcpFrontend.TargetPort = 3000
	c.Assert(saveEngineFrontendRecord(tmpDir, ef), IsNil)

	// Update port and save again (simulating switchover).
	ef.NvmeTcpFrontend.TargetIP = "10.0.0.2"
	ef.NvmeTcpFrontend.TargetPort = 4000
	c.Assert(saveEngineFrontendRecord(tmpDir, ef), IsNil)

	records, err := loadEngineFrontendRecords(tmpDir)
	c.Assert(err, IsNil)
	c.Assert(len(records), Equals, 1)
	c.Assert(records[0].TargetIP, Equals, "10.0.0.2")
	c.Assert(records[0].TargetPort, Equals, int32(4000))
}

// --- RecoverFromHost tests ---

func (s *TestSuite) TestRecoverFromHostNvmfReconstructsEndpoint(c *C) {
	updateCh := make(chan interface{}, 1)

	ef := NewEngineFrontend("ef-nvmf-recover", "engine-r", "vol-r",
		lhtypes.FrontendSPDKTCPNvmf, 1048576, 0, 0, updateCh)
	ef.EngineIP = "10.0.0.5"
	ef.NvmeTcpFrontend.TargetIP = "10.0.0.5"
	ef.NvmeTcpFrontend.TargetPort = 4420

	err := ef.RecoverFromHost(nil)
	c.Assert(err, IsNil)

	got := ef.Get()
	c.Assert(got.State, Equals, string(lhtypes.InstanceStateRunning))

	expectedNqn := helpertypes.GetNQN("engine-r")
	expectedEndpoint := GetNvmfEndpoint(expectedNqn, "10.0.0.5", 4420)
	c.Assert(got.Endpoint, Equals, expectedEndpoint)
	c.Assert(got.TargetIp, Equals, "10.0.0.5")
	c.Assert(got.TargetPort, Equals, int32(4420))

	// Drain the update channel.
	select {
	case <-updateCh:
	case <-time.After(time.Second):
		c.Fatal("expected update on UpdateCh")
	}
}

func (s *TestSuite) TestRecoverFromHostNvmfNoPortLeavesEmptyEndpoint(c *C) {
	updateCh := make(chan interface{}, 1)

	ef := NewEngineFrontend("ef-nvmf-noport", "engine-np", "vol-np",
		lhtypes.FrontendSPDKTCPNvmf, 1048576, 0, 0, updateCh)
	ef.EngineIP = "10.0.0.6"
	// TargetPort is 0 (not recovered from old record).

	err := ef.RecoverFromHost(nil)
	c.Assert(err, IsNil)

	got := ef.Get()
	c.Assert(got.State, Equals, string(lhtypes.InstanceStateRunning))
	// Endpoint should remain empty because port is 0.
	c.Assert(got.Endpoint, Equals, "")
	c.Assert(got.TargetIp, Equals, "10.0.0.6")
	c.Assert(got.TargetPort, Equals, int32(0))
}

func (s *TestSuite) TestRecoverFromHostEmptyFrontend(c *C) {
	updateCh := make(chan interface{}, 1)

	ef := NewEngineFrontend("ef-empty", "engine-e", "vol-e",
		lhtypes.FrontendEmpty, 1024, 0, 0, updateCh)

	err := ef.RecoverFromHost(nil)
	c.Assert(err, IsNil)

	got := ef.Get()
	c.Assert(got.State, Equals, string(lhtypes.InstanceStateRunning))
	c.Assert(got.Endpoint, Equals, "")
}

func (s *TestSuite) TestRecoverFromHostRejectsNonPendingState(c *C) {
	updateCh := make(chan interface{}, 1)

	ef := NewEngineFrontend("ef-running", "engine-run", "vol-run",
		lhtypes.FrontendSPDKTCPNvmf, 1024, 0, 0, updateCh)
	ef.State = lhtypes.InstanceStateRunning

	err := ef.RecoverFromHost(nil)
	c.Assert(err, NotNil)
	c.Assert(err.Error(), Matches, ".*invalid state.*")
}

func (s *TestSuite) TestRecoverFromHostUnsupportedFrontendSetsError(c *C) {
	updateCh := make(chan interface{}, 1)

	ef := NewEngineFrontend("ef-unknown", "engine-u", "vol-u",
		"unknown-frontend", 1024, 0, 0, updateCh)

	err := ef.RecoverFromHost(nil)
	c.Assert(err, NotNil)

	got := ef.Get()
	c.Assert(got.State, Equals, string(lhtypes.InstanceStateError))
	c.Assert(got.ErrorMsg, Matches, ".*unsupported frontend type.*")
}

// --- recoverEngineFrontends integration test ---

func (s *TestSuite) TestRecoverEngineFrontendsRestoresTargetIPAndPort(c *C) {
	tmpDir := c.MkDir()

	// Persist a record.
	ef := NewEngineFrontend("ef-int", "engine-int", "vol-int",
		lhtypes.FrontendSPDKTCPNvmf, 1048576, 0, 0, make(chan interface{}, 1))
	ef.NvmeTcpFrontend.TargetIP = "10.0.0.10"
	ef.NvmeTcpFrontend.TargetPort = 5555
	c.Assert(saveEngineFrontendRecord(tmpDir, ef), IsNil)

	// Load records and build EngineFrontend — simulating what recoverEngineFrontends does.
	records, err := loadEngineFrontendRecords(tmpDir)
	c.Assert(err, IsNil)
	c.Assert(len(records), Equals, 1)

	record := records[0]
	updateCh := make(chan interface{}, 10)
	recovered := NewEngineFrontend(record.Name, record.EngineName, record.VolumeName,
		record.Frontend, record.SpecSize, 0, 0, updateCh)
	if recovered.NvmeTcpFrontend != nil {
		if record.TargetIP != "" {
			recovered.NvmeTcpFrontend.TargetIP = record.TargetIP
			recovered.EngineIP = record.TargetIP
		}
		if record.TargetPort != 0 {
			recovered.NvmeTcpFrontend.TargetPort = record.TargetPort
		}
	}

	// Now RecoverFromHost should reconstruct proper endpoint.
	c.Assert(recovered.RecoverFromHost(nil), IsNil)

	got := recovered.Get()
	c.Assert(got.State, Equals, string(lhtypes.InstanceStateRunning))

	expectedNqn := helpertypes.GetNQN("engine-int")
	expectedEndpoint := GetNvmfEndpoint(expectedNqn, "10.0.0.10", 5555)
	c.Assert(got.Endpoint, Equals, expectedEndpoint)
	c.Assert(got.TargetIp, Equals, "10.0.0.10")
	c.Assert(got.TargetPort, Equals, int32(5555))
}

// --- Empty directory with no record file ---

func (s *TestSuite) TestLoadRecordsSkipsDirWithoutRecordFile(c *C) {
	tmpDir := c.MkDir()

	// Create an empty volume directory (no enginefrontend.json inside).
	emptyDir := filepath.Join(tmpDir, engineFrontendSubDir, "vol-empty")
	c.Assert(os.MkdirAll(emptyDir, 0700), IsNil)

	records, err := loadEngineFrontendRecords(tmpDir)
	c.Assert(err, IsNil)
	c.Assert(len(records), Equals, 0)

	// The empty directory should still exist (it's not corrupted, just missing).
	_, statErr := os.Stat(emptyDir)
	c.Assert(statErr, IsNil)
}
</file>

<file path="pkg/spdk/enginefrontend_persist.go">
package spdk

import (
	"encoding/json"
	"fmt"
	"os"
	"path/filepath"
	"time"

	"github.com/sirupsen/logrus"

	"github.com/longhorn/longhorn-spdk-engine/pkg/types"
)

const (
	engineFrontendSubDir  = "enginefrontends"
	engineFrontendRecFile = "enginefrontend.json"
)

// EngineFrontendRecord holds the minimal metadata needed to recover an
// EngineFrontend after an instance-manager restart. It is persisted to
// <metadataDir>/enginefrontends/<volumeName>/enginefrontend.json.
type EngineFrontendRecord struct {
	Name       string `json:"name"`
	EngineName string `json:"engineName"`
	VolumeName string `json:"volumeName"`
	Frontend   string `json:"frontend"`
	SpecSize   uint64 `json:"specSize"`
	TargetIP   string `json:"targetIP"`
	TargetPort int32  `json:"targetPort"`
}

// engineFrontendRecordDir returns the directory path for a volume's record.
func engineFrontendRecordDir(metadataDir, volumeName string) string {
	return filepath.Join(metadataDir, engineFrontendSubDir, volumeName)
}

// engineFrontendRecordPath returns the full file path for a volume's record.
func engineFrontendRecordPath(metadataDir, volumeName string) string {
	return filepath.Join(engineFrontendRecordDir(metadataDir, volumeName), engineFrontendRecFile)
}

// saveEngineFrontendRecord persists the engine frontend metadata to disk.
// It writes to a temporary file first and then renames for atomicity.
func saveEngineFrontendRecord(metadataDir string, ef *EngineFrontend) error {
	if metadataDir == "" {
		return nil
	}

	// UBLK frontends cannot be recovered after restart, so skip persistence.
	if types.IsUblkFrontend(ef.Frontend) {
		return nil
	}

	var targetIP string
	var targetPort int32
	if ef.NvmeTcpFrontend != nil {
		targetIP = ef.NvmeTcpFrontend.TargetIP
		targetPort = ef.NvmeTcpFrontend.TargetPort
	}

	record := &EngineFrontendRecord{
		Name:       ef.Name,
		EngineName: ef.EngineName,
		VolumeName: ef.VolumeName,
		Frontend:   ef.Frontend,
		SpecSize:   ef.SpecSize,
		TargetIP:   targetIP,
		TargetPort: targetPort,
	}

	dir := engineFrontendRecordDir(metadataDir, ef.VolumeName)
	if err := os.MkdirAll(dir, 0700); err != nil {
		return fmt.Errorf("failed to create engine frontend record directory %s: %w", dir, err)
	}

	data, err := json.MarshalIndent(record, "", "  ")
	if err != nil {
		return fmt.Errorf("failed to marshal engine frontend record for %s: %w", ef.Name, err)
	}

	targetPath := engineFrontendRecordPath(metadataDir, ef.VolumeName)
	tmpPath := targetPath + ".tmp"

	if err := os.WriteFile(tmpPath, data, 0600); err != nil {
		return fmt.Errorf("failed to write engine frontend record temp file %s: %w", tmpPath, err)
	}

	if err := os.Rename(tmpPath, targetPath); err != nil {
		// Best effort cleanup of temp file.
		if errRemove := os.Remove(tmpPath); errRemove != nil {
			logrus.WithError(errRemove).Warnf("Failed to remove engine frontend record temp file %s", tmpPath)
		}
		return fmt.Errorf("failed to rename engine frontend record %s -> %s: %w", tmpPath, targetPath, err)
	}

	return nil
}

// removeEngineFrontendRecord removes the persisted engine frontend record
// for the given volume name.
func removeEngineFrontendRecord(metadataDir, volumeName string) error {
	if metadataDir == "" {
		return nil
	}

	dir := engineFrontendRecordDir(metadataDir, volumeName)
	if err := os.RemoveAll(dir); err != nil {
		return fmt.Errorf("failed to remove engine frontend record directory %s: %w", dir, err)
	}

	return nil
}

// loadEngineFrontendRecords scans the engine frontend records directory
// and returns all valid records. Invalid or corrupted records are logged
// and skipped.
func loadEngineFrontendRecords(metadataDir string) ([]*EngineFrontendRecord, error) {
	if metadataDir == "" {
		return nil, nil
	}

	baseDir := filepath.Join(metadataDir, engineFrontendSubDir)

	var entries []os.DirEntry
	var readErr error
	for attempt := 0; attempt < 3; attempt++ {
		entries, readErr = os.ReadDir(baseDir)
		if readErr == nil {
			break
		}
		if os.IsNotExist(readErr) {
			return nil, nil
		}
		logrus.WithError(readErr).Warnf("Failed to read engine frontend records directory %s (attempt %d/3)", baseDir, attempt+1)
		time.Sleep(500 * time.Millisecond)
	}
	if readErr != nil {
		return nil, fmt.Errorf("failed to read engine frontend records directory %s after retries: %w", baseDir, readErr)
	}

	var records []*EngineFrontendRecord
	for _, entry := range entries {
		if !entry.IsDir() {
			continue
		}

		volumeName := entry.Name()
		recordPath := engineFrontendRecordPath(metadataDir, volumeName)

		data, err := os.ReadFile(recordPath)
		if err != nil {
			if os.IsNotExist(err) {
				logrus.Warnf("Engine frontend record directory %s exists but has no %s, skipping", volumeName, engineFrontendRecFile)
			} else {
				logrus.WithError(err).Warnf("Failed to read engine frontend record %s, skipping", recordPath)
			}
			continue
		}

		record := &EngineFrontendRecord{}
		if err := json.Unmarshal(data, record); err != nil {
			logrus.WithError(err).Warnf("Failed to parse engine frontend record %s, removing corrupted record", recordPath)
			if removeErr := os.RemoveAll(filepath.Join(baseDir, volumeName)); removeErr != nil {
				logrus.WithError(removeErr).Warnf("Failed to remove corrupted engine frontend record directory %s", volumeName)
			}
			continue
		}

		if record.Name == "" || record.VolumeName == "" {
			logrus.Warnf("Engine frontend record %s has empty name or volume name, removing invalid record", recordPath)
			if removeErr := os.RemoveAll(filepath.Join(baseDir, volumeName)); removeErr != nil {
				logrus.WithError(removeErr).Warnf("Failed to remove invalid engine frontend record directory %s", volumeName)
			}
			continue
		}

		records = append(records, record)
	}

	return records, nil
}
</file>

<file path="pkg/spdk/enginefrontend_race_test.go">
package spdk

import (
	"fmt"
	"sync"

	lhtypes "github.com/longhorn/longhorn-spdk-engine/pkg/types"

	. "gopkg.in/check.v1"
)

func (s *TestSuite) TestEngineFrontendResumeConcurrentWithValidateAndUpdate(c *C) {
	fmt.Println("Testing EngineFrontend.Resume concurrent with ValidateAndUpdate")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendEmpty, 1024, 0, 0, make(chan interface{}, 4096))
	ef.State = lhtypes.InstanceStateRunning

	const iterations = 200

	errCh := make(chan error, 2)
	var wg sync.WaitGroup
	wg.Add(2)

	go func() {
		defer wg.Done()
		for i := 0; i < iterations; i++ {
			if err := ef.Resume(nil); err != nil {
				errCh <- err
				return
			}
		}
	}()

	go func() {
		defer wg.Done()
		for i := 0; i < iterations; i++ {
			if err := ef.ValidateAndUpdate(nil); err != nil {
				errCh <- err
				return
			}
		}
	}()

	wg.Wait()
	close(errCh)

	for err := range errCh {
		c.Assert(err, IsNil)
	}

	c.Assert(string(ef.State), Equals, string(lhtypes.InstanceStateRunning))
	c.Assert(ef.ErrorMsg, Equals, "")
}
</file>

<file path="pkg/spdk/enginefrontend.go">
package spdk

import (
	"context"
	"fmt"
	"net"
	"runtime/debug"
	"strconv"
	"strings"
	"sync"
	"time"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"go.uber.org/multierr"

	"github.com/longhorn/longhorn-spdk-engine/pkg/client"
	"github.com/longhorn/longhorn-spdk-engine/pkg/types"
	"github.com/longhorn/longhorn-spdk-engine/pkg/util"

	safelog "github.com/longhorn/longhorn-spdk-engine/pkg/log"

	"github.com/longhorn/go-spdk-helper/pkg/initiator"
	"github.com/longhorn/types/pkg/generated/spdkrpc"

	spdkclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
	helpertypes "github.com/longhorn/go-spdk-helper/pkg/types"
)

type EngineFrontend struct {
	sync.RWMutex

	Name       string
	EngineName string
	VolumeName string
	SpecSize   uint64
	ActualSize uint64

	Frontend string
	Endpoint string

	EngineIP string

	NvmeTcpFrontend *NvmeTcpFrontend
	UblkFrontend    *UblkFrontend

	State    types.InstanceState
	ErrorMsg string

	initiator      *initiator.Initiator
	dmDeviceIsBusy bool

	IsRestoring           bool
	RestoringSnapshotName string

	isCreating            bool
	isSwitchingOver       bool
	isExpanding           bool
	lastExpansionFailedAt string
	lastExpansionError    string

	// UpdateCh should not be protected by the engine lock
	UpdateCh chan interface{}

	// stopCh is closed when the engine frontend is deleted to signal
	// background goroutines to abort early.
	stopCh chan struct{}

	// Test hook for switchover target engine name resolution.
	resolveEngineNameByTargetAddressFn func(targetAddress string) (string, error)
	// Test hook for switchover target connect/rollback.
	startNvmeTCPInitiatorFn func(transportAddress, transportServiceID string, dmDeviceAndEndpointCleanupRequired bool, stop bool) (dmDeviceIsBusy bool, err error)
	// Test hook for endpoint retrieval after switchover.
	getInitiatorEndpointFn func() string

	// metadataDir is the base path for persisting engine frontend records.
	// If empty, persistence is disabled.
	metadataDir string

	log *safelog.SafeLogger
}

type NvmeTcpFrontend struct {
	TargetIP   string
	TargetPort int32

	Nqn   string
	Nguid string
}

type UblkFrontend struct {
	// spec
	UblkQueueDepth    int32
	UblkNumberOfQueue int32

	// status
	UblkID int32
}

func getUblkQueueDepth(ublkQueueDepth int32) int32 {
	if ublkQueueDepth == 0 {
		return types.DefaultUblkQueueDepth
	}
	return ublkQueueDepth
}

func getUblkNumberOfQueue(ublkNumberOfQueue int32) int32 {
	if ublkNumberOfQueue == 0 {
		return types.DefaultUblkNumberOfQueue
	}
	return ublkNumberOfQueue
}

func NewEngineFrontend(engineFrontendName, engineName, volumeName, frontend string, specSize uint64, ublkQueueDepth, ublkNumberOfQueue int32,
	engineFrontendUpdateCh chan interface{}) *EngineFrontend {
	log := logrus.StandardLogger().WithFields(logrus.Fields{
		"engineFrontendName": engineFrontendName,
		"engineName":         engineName,
		"volumeName":         volumeName,
		"frontend":           frontend,
		"specSize":           specSize,
	})

	if types.IsUblkFrontend(frontend) {
		log = log.WithFields(logrus.Fields{
			"ublkQueueDepth":    getUblkQueueDepth(ublkQueueDepth),
			"ublkNumberOfQueue": getUblkNumberOfQueue(ublkNumberOfQueue),
		})
	}

	roundedSpecSize := util.RoundUp(specSize, helpertypes.MiB)
	if roundedSpecSize != specSize {
		log.Infof("Rounded up spec size from %v to %v since the spec size should be multiple of MiB", specSize, roundedSpecSize)
		log = log.WithField("roundedSpecSize", roundedSpecSize)
	}

	nvmeTcpFrontend := &NvmeTcpFrontend{}
	ublkFrontend := &UblkFrontend{
		UblkQueueDepth:    getUblkQueueDepth(ublkQueueDepth),
		UblkNumberOfQueue: getUblkNumberOfQueue(ublkNumberOfQueue),
	}

	return &EngineFrontend{
		Name:       engineFrontendName,
		EngineName: engineName,
		VolumeName: volumeName,
		SpecSize:   specSize,

		Frontend: frontend,

		NvmeTcpFrontend: nvmeTcpFrontend,
		UblkFrontend:    ublkFrontend,

		State:    types.InstanceStatePending,
		ErrorMsg: "",

		UpdateCh: engineFrontendUpdateCh,
		stopCh:   make(chan struct{}),
		log:      safelog.NewSafeLogger(log),
	}
}

// Create creates the engine frontend. On failure, it sets the frontend state
// to InstanceStateError with the error message and returns the error so callers
// can surface the attach failure instead of treating it as a successful start.
func (ef *EngineFrontend) Create(spdkClient *spdkclient.Client, targetAddress string) (ret *spdkrpc.EngineFrontend, err error) {
	ef.log.WithFields(logrus.Fields{
		"targetAddress": targetAddress,
		"frontend":      ef.Name,
	}).Info("Creating engine frontend")

	targetIP, _, err := splitHostPort(targetAddress)
	if err != nil {
		return nil, errors.Mark(
			errors.Wrapf(err, "failed to split target address %v", targetAddress),
			ErrEngineFrontendCreateInvalidArgument)
	}

	// Phase 1: Acquire lock to check state and establish the isCreating guard
	ef.Lock()
	if ef.State != types.InstanceStatePending {
		ef.Unlock()
		return nil, errors.Wrapf(ErrEngineFrontendCreatePrecondition, "invalid state %s for engine frontend %s creation", ef.State, ef.Name)
	}
	if ef.isCreating {
		ef.Unlock()
		return nil, errors.Wrapf(ErrEngineFrontendCreatePrecondition, "engine frontend %s is already creating", ef.Name)
	}
	ef.isCreating = true
	ef.EngineIP = targetIP
	ef.Unlock()

	var requireUpdate bool
	var frontendErr error

	// Phase 3: Cleanup and final state resolution
	defer func() {
		if r := recover(); r != nil {
			ef.log.WithFields(logrus.Fields{
				"panic": string(debug.Stack()),
			}).Errorf("Recovered panic during engine frontend %s creation: %v", ef.Name, r)
			frontendErr = errors.Wrapf(fmt.Errorf("%v", r), "panic during engine frontend %s creation", ef.Name)
		}

		ef.Lock()

		ef.isCreating = false

		if frontendErr != nil {
			ef.log.WithError(frontendErr).Errorf("Failed to create engine frontend %s", ef.Name)
			if ef.State != types.InstanceStateError {
				ef.State = types.InstanceStateError
				requireUpdate = true
			}
			ef.ErrorMsg = frontendErr.Error()
			ret = nil
			err = frontendErr
		} else {
			if ef.State != types.InstanceStateError {
				ef.ErrorMsg = ""
			}
			if ef.State != types.InstanceStateRunning {
				ef.State = types.InstanceStateRunning
				requireUpdate = true
			}
			ef.log.Info("Created engine frontend")

			// Persist record AFTER successful creation.
			if err := saveEngineFrontendRecord(ef.metadataDir, ef); err != nil {
				ef.log.WithError(err).Warn("Failed to persist engine frontend record")
			}

			ret = ef.getWithoutLock()
		}
		ef.Unlock()

		if requireUpdate {
			ef.UpdateCh <- nil
		}
	}()

	// Phase 2: Operations without lock
	initiatorCreationRequired, err := ef.isInitiatorCreationRequired(targetIP)
	if err != nil {
		frontendErr = err
		return
	}
	if !initiatorCreationRequired {
		return
	}

	ef.log.UpdateLoggerWithWarnOnFailure(logrus.Fields{
		"initiatorCreationRequired": initiatorCreationRequired,
		"targetAddress":             targetAddress,
	}, "Failed to update logger with initiatorCreationRequired and targetAddress during engine creation")

	ef.log.Info("Handling frontend during engine frontend creation")

	if frontendErr = ef.handleFrontend(spdkClient, targetAddress); frontendErr != nil {
		return
	}

	return
}

func (ef *EngineFrontend) Delete(spdkClient *spdkclient.Client) (err error) {
	requireUpdate := false

	ef.Lock()
	if ef.isCreating {
		ef.Unlock()
		return errors.Wrapf(ErrEngineFrontendLifecyclePrecondition, "engine frontend %s is still creating", ef.Name)
	}
	if ef.isSwitchingOver {
		ef.Unlock()
		return errors.Wrapf(ErrSwitchOverTargetPrecondition, "engine frontend %s is switching over target", ef.Name)
	}
	if ef.isExpanding {
		ef.Unlock()
		return errors.Wrapf(ErrEngineFrontendLifecyclePrecondition, "engine frontend %s is still expanding", ef.Name)
	}

	defer func() {
		// Considering that there may be still pending validations, it's better to update the state after the deletion.
		if err != nil {
			ef.log.WithError(err).Errorf("Failed to delete engine frontend")
			if ef.State != types.InstanceStateError {
				ef.State = types.InstanceStateError
				ef.ErrorMsg = err.Error()
				requireUpdate = true
			}
		} else {
			if ef.State != types.InstanceStateError {
				ef.ErrorMsg = ""
			}
		}
		if ef.State != types.InstanceStateError && ef.State != types.InstanceStateTerminating && ef.State != types.InstanceStateStopped {
			ef.State = types.InstanceStateTerminating
			requireUpdate = true
		}

		ef.Unlock()

		if requireUpdate {
			ef.UpdateCh <- nil
		}
	}()

	// Signal background goroutines to stop.
	select {
	case <-ef.stopCh:
		// Already closed.
	default:
		close(ef.stopCh)
	}

	ef.log.WithField("hasInitiator", ef.initiator != nil).Info("Deleting engine frontend")

	if ef.initiator != nil {
		if _, err := ef.initiator.Stop(spdkClient, true, true, true); err != nil {
			return err
		}
		ef.initiator = nil
		ef.Endpoint = ""

		requireUpdate = true
	}

	if ef.NvmeTcpFrontend != nil {
		ef.NvmeTcpFrontend.TargetIP = ""
		ef.NvmeTcpFrontend.TargetPort = 0
		ef.NvmeTcpFrontend.Nqn = ""
		ef.NvmeTcpFrontend.Nguid = ""
	}

	ef.log.Info("Deleted engine frontend")

	// Remove persisted record AFTER successful deletion.
	if err := removeEngineFrontendRecord(ef.metadataDir, ef.VolumeName); err != nil {
		ef.log.WithError(err).Warn("Failed to remove engine frontend record")
	}

	return nil
}

func (ef *EngineFrontend) handleFrontend(spdkClient *spdkclient.Client, targetAddress string) (err error) {
	switch ef.Frontend {
	case types.FrontendEmpty:
		ef.log.Info("No frontend specified, will not expose bdev for engine")
		return nil
	case types.FrontendUBLK:
		ef.log.Infof("Handling ublk frontend for engine frontend creation, targetAddress: %s", targetAddress)
		return ef.createUblkFrontend(spdkClient)
	case types.FrontendSPDKTCPBlockdev, types.FrontendSPDKTCPNvmf:
		ef.log.Infof("Handling NVMe/TCP frontend for engine frontend creation, targetAddress: %s", targetAddress)
		return ef.createNvmeTcpFrontend(spdkClient, targetAddress)
	default:
		return fmt.Errorf("unknown frontend type %s", ef.Frontend)
	}
}

func (ef *EngineFrontend) createNvmeTcpFrontend(spdkClient *spdkclient.Client, targetAddress string) (err error) {
	if ef.NvmeTcpFrontend == nil {
		return fmt.Errorf("failed to create NVMe/TCP frontend: invalid NvmeTcpFrontend: %v", ef.NvmeTcpFrontend)
	}

	targetIP, targetPort, err := splitHostPort(targetAddress)
	if err != nil {
		return errors.Wrapf(err, "failed to split host port for engine frontend %v", ef.Name)
	}

	frontendConfigured := ef.isNvmeTcpFrontendConfigured()

	var i *initiator.Initiator
	if frontendConfigured {
		// If the NVMe frontend is already configured, reuse the existing initiator and connection info.
		ef.log.Infof("Reusing existing initiator. TargetIP: %s, TargetPort: %d", ef.NvmeTcpFrontend.TargetIP, ef.NvmeTcpFrontend.TargetPort)

		i = ef.initiator
	} else {
		ef.log.Infof("Creating new initiator for NVMe/TCP frontend: targetAddress: %s", targetAddress)

		var nqn, nguid string
		i, nqn, nguid, err = ef.newNvmeTcpInitiator()
		if err != nil {
			return errors.Wrap(err, "failed to create NVMe/TCP initiator")
		}

		ef.Lock()
		ef.NvmeTcpFrontend.Nqn = nqn
		ef.NvmeTcpFrontend.Nguid = nguid
		ef.Unlock()
	}

	dmDeviceIsBusy := false

	defer func() {
		if err != nil {
			return
		}

		if !frontendConfigured {
			ef.Lock()
			switch ef.Frontend {
			case types.FrontendSPDKTCPBlockdev:
				ef.NvmeTcpFrontend.TargetIP = targetIP
				ef.NvmeTcpFrontend.TargetPort = targetPort
				ef.Endpoint = i.GetEndpoint()
				ef.initiator = i
				ef.dmDeviceIsBusy = dmDeviceIsBusy
			case types.FrontendSPDKTCPNvmf:
				ef.NvmeTcpFrontend.TargetIP = targetIP
				ef.NvmeTcpFrontend.TargetPort = targetPort
				ef.Endpoint = GetNvmfEndpoint(ef.NvmeTcpFrontend.Nqn, targetIP, targetPort)
			}
			endpoint := ef.Endpoint
			targetPortForLog := ef.NvmeTcpFrontend.TargetPort
			ef.Unlock()
			ef.log.UpdateLoggerWithWarnOnFailure(logrus.Fields{
				"endpoint":   endpoint,
				"targetPort": targetPortForLog,
			}, "Failed to update logger with endpoint and port during engine frontend handling")
		}

		ef.log.Infof("Created engine frontend")
	}()

	if ef.Frontend == types.FrontendSPDKTCPNvmf {
		return nil
	}

	// In the expansion flow we MUST NOT disconnect the NVMe target.
	//
	// Technical Reason:
	// 1. If we disconnect while the dm-linear device is still active (suspended but open),
	//    the Linux kernel cannot fully release the /dev/nvmeXnX resource, leading to
	//    a "zombie" device node.
	// 2. When we reconnect later, the kernel will detect a naming conflict and assign
	//    a new, non-healthy device name (e.g., nvme3n2) instead of reusing nvme1n1.
	// 3. By skipping disconnect, the kernel keeps the existing controller session in
	//    a "reconnecting" state. Once SPDK re-exposes the resized RAID, the kernel
	//    automatically recovers the original path (nvme1n1) and perceives the new size,
	//    allowing a successful 'dmsetup reload' without breaking the mount point.
	ef.RLock()
	disconnectTarget := !ef.isExpanding
	ef.RUnlock()

	dmDeviceIsBusy, err = i.StartNvmeTCPInitiator(targetIP, strconv.Itoa(int(targetPort)), true, disconnectTarget)
	if err != nil {
		return errors.Wrapf(err, "failed to start NVMe/TCP initiator for engine frontend %v", ef.Name)
	}

	return nil
}

func (ef *EngineFrontend) newNvmeTcpInitiator() (i *initiator.Initiator, nqn, nguid string, err error) {
	nqn = helpertypes.GetNQN(ef.EngineName)
	nguid = generateNGUID(ef.EngineName)

	nvmeTCPInfo := &initiator.NVMeTCPInfo{
		SubsystemNQN: nqn,
	}
	i, err = initiator.NewInitiator(ef.VolumeName, initiator.HostProc, nvmeTCPInfo, nil)
	if err != nil {
		return i, "", "", errors.Wrapf(err, "failed to create NVMe/TCP initiator for engine %v", ef.Name)
	}

	return i, nqn, nguid, nil
}

func (ef *EngineFrontend) getWithoutLock() (res *spdkrpc.EngineFrontend) {
	res = &spdkrpc.EngineFrontend{
		Name:       ef.Name,
		VolumeName: ef.VolumeName,
		EngineName: ef.EngineName,
		SpecSize:   ef.SpecSize,
		ActualSize: ef.ActualSize,
		Frontend:   ef.Frontend,
		Endpoint:   ef.Endpoint,

		State:                 string(ef.State),
		ErrorMsg:              ef.ErrorMsg,
		IsExpanding:           ef.isExpanding,
		LastExpansionError:    ef.lastExpansionError,
		LastExpansionFailedAt: ef.lastExpansionFailedAt,
	}

	if ef.NvmeTcpFrontend != nil {
		res.TargetIp = ef.NvmeTcpFrontend.TargetIP
		res.TargetPort = ef.NvmeTcpFrontend.TargetPort
	}

	if ef.UblkFrontend != nil {
		res.UblkId = ef.UblkFrontend.UblkID
	}

	return res
}

// SetErrorState sets the engine frontend to error state.
func (ef *EngineFrontend) SetErrorState() {
	needUpdate := false

	ef.Lock()
	defer func() {
		ef.Unlock()

		if needUpdate {
			ef.UpdateCh <- nil
		}
	}()

	if ef.State != types.InstanceStateStopped && ef.State != types.InstanceStateError {
		ef.log.Error("Setting engine frontend to error state")
		ef.State = types.InstanceStateError
		needUpdate = true
	}
}

// Get returns a copy of the current engine frontend state.
func (ef *EngineFrontend) Get() (res *spdkrpc.EngineFrontend) {
	ef.RLock()
	defer ef.RUnlock()

	return ef.getWithoutLock()
}

func (ef *EngineFrontend) isNvmeTcpFrontendConfigured() bool {
	ef.RLock()
	defer ef.RUnlock()

	if ef.NvmeTcpFrontend == nil || ef.initiator == nil {
		return false
	}

	if len(ef.NvmeTcpFrontend.TargetIP) == 0 || ef.NvmeTcpFrontend.TargetPort == 0 ||
		len(ef.NvmeTcpFrontend.Nqn) == 0 || len(ef.NvmeTcpFrontend.Nguid) == 0 {
		return false
	}

	return true
}

func (ef *EngineFrontend) createUblkFrontend(spdkClient *spdkclient.Client) (err error) {
	if ef.UblkFrontend == nil {
		return fmt.Errorf("failed to createUblkFrontend: invalid UblkFrontend: %v", ef.UblkFrontend)
	}
	dmDeviceIsBusy := false

	ublkInfo := &initiator.UblkInfo{
		BdevName:          ef.EngineName,
		UblkQueueDepth:    ef.UblkFrontend.UblkQueueDepth,
		UblkNumberOfQueue: ef.UblkFrontend.UblkNumberOfQueue,

		UblkID: initiator.UnInitializedUblkId,
	}
	i, err := initiator.NewInitiator(ef.VolumeName, initiator.HostProc, nil, ublkInfo)
	if err != nil {
		return errors.Wrapf(err, "failed to create initiator for engine %v", ef.Name)
	}

	defer func() {
		if err == nil {
			ef.Lock()
			ef.initiator = i
			ef.dmDeviceIsBusy = dmDeviceIsBusy
			ef.Endpoint = i.GetEndpoint()
			endpoint := ef.Endpoint
			ublkID := ef.UblkFrontend.UblkID
			ef.Unlock()

			ef.log.UpdateLoggerWithWarnOnFailure(logrus.Fields{
				"endpoint": endpoint,
				"ublkID":   ublkID,
			}, "Failed to update logger with endpoint and port during engine creation")
			ef.log.Infof("Created engine frontend")
		}
	}()

	dmDeviceIsBusy, err = i.StartUblkInitiator(spdkClient, true)
	if err != nil {
		return errors.Wrapf(err, "failed to start initiator for engine %v", ef.Name)
	}

	ef.Lock()
	ef.UblkFrontend.UblkID = i.UblkInfo.UblkID
	ef.Unlock()

	return nil
}

func (ef *EngineFrontend) isInitiatorCreationRequired(targetIP string) (bool, error) {
	if types.IsUblkFrontend(ef.Frontend) {
		return true, nil
	}

	if ef.NvmeTcpFrontend == nil {
		return false, fmt.Errorf("failed to isInitiatorCreationRequired: invalid NvmeTcpFrontend: %v", ef.NvmeTcpFrontend)
	}

	return ef.NvmeTcpFrontend.TargetPort == 0, nil
}

// Expand performs an online volume expansion for the Longhorn Engine using SPDK.
// It expands the underlying replica logical volumes (lvol), recreates the SPDK RAID bdev,
// suspends and resumes frontend I/O as needed, and ensures cleanup and status updates on failure.
func (ef *EngineFrontend) Expand(ctx context.Context, spdkClient *spdkclient.Client, size uint64) (retErr error) {
	ef.log.Info("Expanding engine frontend")

	// Phase 1: Acquire lock to read state and check expansion guards.
	ef.Lock()
	if ef.isCreating {
		ef.Unlock()
		return fmt.Errorf("engine frontend %s is still creating", ef.Name)
	}
	if ef.isSwitchingOver {
		ef.Unlock()
		return errors.Wrapf(ErrSwitchOverTargetPrecondition, "engine frontend %s is switching over target", ef.Name)
	}

	originalSize := ef.SpecSize
	engineIP := ef.EngineIP
	engineName := ef.EngineName
	frontend := ef.Frontend

	var targetAddress string
	if ef.NvmeTcpFrontend != nil {
		targetAddress = net.JoinHostPort(ef.NvmeTcpFrontend.TargetIP, strconv.Itoa(int(ef.NvmeTcpFrontend.TargetPort)))
	}

	engineSpdkClient, err := GetServiceClient(net.JoinHostPort(engineIP, strconv.Itoa(types.SPDKServicePort)))
	if err != nil {
		ef.Unlock()
		return errors.Wrapf(err, "failed to get SPDK client to expand engine frontend %v", ef.Name)
	}
	defer func() {
		if errClose := engineSpdkClient.Close(); errClose != nil {
			ef.log.WithError(errClose).Error("Failed to close SPDK client")
		}
	}()

	requireExpansion, err := ef.requireExpansion(ctx, engineSpdkClient, size)
	if err != nil {
		ef.Unlock()
		return errors.Wrap(err, "failed to check whether expansion is required")
	}
	ef.Unlock()
	// Phase 1 ends: lock released.

	// engineErr will be set when the engine failed to do any non-recoverable operations.
	expanded := false
	backendExpansionError := ""
	backendExpansionFailedAt := ""
	var engineActualSize uint64

	defer func() {
		if r := recover(); r != nil {
			ef.log.WithFields(logrus.Fields{
				"panic": string(debug.Stack()),
			}).Errorf("Recovered panic during engine frontend %s expansion: %v", ef.Name, r)
			retErr = errors.Wrapf(fmt.Errorf("%v", r), "panic during engine frontend %s expansion", ef.Name)
		}

		// Phase 3: Re-acquire lock to update state.
		ef.Lock()
		ef.finishExpansion(originalSize, expanded, size, retErr, backendExpansionError, backendExpansionFailedAt, engineActualSize)
		ef.Unlock()

		ef.UpdateCh <- nil
	}()

	if !requireExpansion {
		ef.log.Info("No need to expand engine")
		expanded = true
		return nil
	}

	// Phase 2: Long-running operations without lock.
	suspended, err := ef.prepareExpansion()
	if err != nil {
		return errors.Wrap(err, "prepare raid for expansion failed")
	}
	if suspended {
		defer func() {
			if ef.initiator != nil {
				if frontendErr := ef.initiator.Resume(); frontendErr != nil {
					retErr = multierr.Append(retErr, errors.Wrapf(frontendErr, "original error; resume failed"))
				}
			}
		}()
	}

	if err := engineSpdkClient.EngineExpand(ctx, engineName, size); err != nil {
		return errors.Wrapf(err, "failed to expand engine %v", engineName)
	}

	engine, err := engineSpdkClient.EngineGet(engineName)
	if err != nil {
		return errors.Wrapf(err, "failed to get engine %v after expansion", engineName)
	}
	engineActualSize = engine.ActualSize
	if engine.LastExpansionError != "" {
		backendExpansionError = engine.LastExpansionError
		backendExpansionFailedAt = engine.LastExpansionFailedAt
		ef.log.Warnf("Engine %s partially failed to expand to %v; keeping engine frontend size at %v: %v",
			engineName, size, originalSize, backendExpansionError)
		return nil
	}

	if targetAddress != "" {
		if err := ef.handleFrontend(spdkClient, targetAddress); err != nil {
			return errors.Wrap(err, "failed to handle frontend")
		}
	}

	// It waits for the kernel to recognize the new physical NVMe capacity
	// and then reloads the dm table to propagate the size change up to the volume.
	if frontend != types.FrontendEmpty && ef.initiator != nil {
		if err := ef.initiator.SyncDmDeviceSize(size); err != nil {
			ef.log.WithError(err).Warnf("failed to sync linear dm device size during engine %s expansion", engineName)
		}
	}

	ef.log.Info("Expanding engine completed")
	expanded = true

	return nil
}

func (ef *EngineFrontend) requireExpansion(ctx context.Context, engineSpdkClient *client.SPDKClient, size uint64) (requireExpansion bool, err error) {
	if ef.isExpanding {
		return false, fmt.Errorf("%w", ErrExpansionInProgress)
	}

	if ef.IsRestoring {
		return false, fmt.Errorf("%w", ErrRestoringInProgress)
	}

	if ef.SpecSize > size {
		return false, fmt.Errorf("%w: cannot expand engine to a smaller size %v, current size %v", ErrExpansionInvalidSize, size, ef.SpecSize)
	}

	if ef.SpecSize == size {
		// EngineFrontend is already at the requested size. However, for offline
		// expansion the Engine's SpecSize may have been adjusted downward by
		// ValidateAndUpdate to match the actual RAID bdev size (built from
		// unexpanded replicas). Check whether the downstream Engine still needs
		// expansion before skipping.
		engine, err := engineSpdkClient.EngineGet(ef.EngineName)
		if err != nil {
			return false, errors.Wrapf(err, "failed to get engine %v during expansion check", ef.EngineName)
		}
		if engine.SpecSize >= size {
			ef.log.Infof("Engine already at requested size %v, skipping expansion", size)
			return false, nil
		}
		ef.log.Infof("Engine frontend at requested size %v but engine SpecSize is %v, proceeding with expansion", size, engine.SpecSize)
	}

	roundedNewSize := util.RoundUp(size, helpertypes.MiB)
	if roundedNewSize != size {
		return false, fmt.Errorf("%w: rounded up spec size from %v to %v since the spec size should be multiple of MiB",
			ErrExpansionInvalidSize, size, roundedNewSize)
	}

	// Check if the expansion is required for downstream engine and replicas
	if err := engineSpdkClient.EngineExpandPrecheck(ctx, ef.EngineName, size); err != nil {
		return false, errors.Wrapf(err, "failed to precheck engine %v expansion", ef.Name)
	}

	// Mark expansion as in progress
	ef.isExpanding = true
	ef.lastExpansionFailedAt = ""
	ef.lastExpansionError = ""

	return true, nil
}

func (ef *EngineFrontend) finishExpansion(fromSize uint64, expanded bool, size uint64, err error, backendExpansionError, backendExpansionFailedAt string, engineActualSize uint64) {
	// Sync ActualSize from the engine whenever we successfully queried it,
	// regardless of whether the expansion itself succeeded or failed.
	if engineActualSize > 0 {
		ef.ActualSize = engineActualSize
	}

	if err != nil {
		ef.log.WithError(err).Errorf("Engine %s failed to expand from size %v to %v", ef.Name, fromSize, size)
		ef.State = types.InstanceStateError
		ef.ErrorMsg = err.Error()
		ef.lastExpansionError = errors.Wrap(err, "engine failed to expand expansion").Error()
		ef.lastExpansionFailedAt = time.Now().UTC().Format(time.RFC3339Nano)

		ef.log.WithError(err).Errorf("Engine %s failed to expand", ef.Name)
		if expanded {
			// The backend expansion succeeded, but post-expansion frontend handling failed.
			ef.SpecSize = size
			ef.log.Warnf("Expanded from size %v to %v but encountered post-expansion error", fromSize, size)

			// Re-persist even on error so the updated SpecSize survives a restart.
			if err := saveEngineFrontendRecord(ef.metadataDir, ef); err != nil {
				ef.log.WithError(err).Warn("Failed to persist engine frontend record after partial expansion")
			}
		} else {
			ef.log.Infof("Failed to expand from size %v to %v", fromSize, size)
		}
		ef.isExpanding = false
		return
	}

	ef.State = types.InstanceStateRunning
	ef.ErrorMsg = ""
	if backendExpansionError != "" {
		ef.lastExpansionError = backendExpansionError
		if backendExpansionFailedAt != "" {
			ef.lastExpansionFailedAt = backendExpansionFailedAt
		} else {
			ef.lastExpansionFailedAt = time.Now().UTC().Format(time.RFC3339Nano)
		}
		ef.log.Warnf("Partially failed to expand from size %v to %v; keeping engine frontend size at %v: %v",
			fromSize, size, fromSize, backendExpansionError)
		ef.isExpanding = false
		return
	}
	if expanded {
		ef.log.Infof("Succeeded to expand from size %v to %v", fromSize, size)
		ef.SpecSize = size

		// Re-persist the record so the updated SpecSize survives a restart.
		if err := saveEngineFrontendRecord(ef.metadataDir, ef); err != nil {
			ef.log.WithError(err).Warn("Failed to persist engine frontend record after expansion")
		}
	} else {
		ef.log.Infof("Failed to expand from size %v to %v", fromSize, size)
	}

	// Clear stale expansion error on success (err == nil && backendExpansionError == "").
	// A previous partial failure may have left lastExpansionError set.
	ef.lastExpansionError = ""
	ef.lastExpansionFailedAt = ""

	ef.isExpanding = false
}

func (ef *EngineFrontend) prepareExpansion() (engineFrontendSuspended bool, err error) {
	switch ef.Frontend {
	case types.FrontendUBLK:
		return false, fmt.Errorf("not support ublk frontend for expansion for engine %s", ef.Name)
	case types.FrontendSPDKTCPBlockdev:
		if ef.Endpoint != "" {
			ef.log.Info("Suspending engine frontend")
			if err := ef.initiator.Suspend(false, false); err != nil {
				return false, errors.Wrapf(err, "failed to suspend engine frontend %s", ef.Name)
			}
			return true, nil
		}
	}
	return false, nil
}

// SuspendFrontend suspends the engine frontend. IO operations will be suspended.
func (ef *EngineFrontend) Suspend(_ *spdkclient.Client) (err error) {
	ef.Lock()
	defer func() {
		if err != nil {
			if ef.State != types.InstanceStateError {
				ef.log.WithError(err).Warn("Failed to suspend engine frontend")
				// Engine is still alive and running and is not really in error state.
				// longhorn-manager will retry to suspend the engine.
			}
			ef.ErrorMsg = err.Error()
		} else {
			ef.State = types.InstanceStateSuspended
			ef.ErrorMsg = ""

			ef.log.Info("Suspended engine frontend")
		}

		ef.Unlock()

		ef.UpdateCh <- nil
	}()

	if ef.State == types.InstanceStateSuspended {
		return nil
	}
	if ef.isSwitchingOver {
		return errors.Wrapf(ErrSwitchOverTargetPrecondition, "engine frontend %s is switching over target", ef.Name)
	}

	ef.log.Info("Suspending engine frontend")

	switch ef.Frontend {
	case types.FrontendSPDKTCPBlockdev:
		nvmeTCPInfo := &initiator.NVMeTCPInfo{
			SubsystemNQN: ef.NvmeTcpFrontend.Nqn,
		}
		i, err := initiator.NewInitiator(ef.VolumeName, initiator.HostProc, nvmeTCPInfo, nil)
		if err != nil {
			return errors.Wrapf(err, "failed to create initiator for suspending engine %s", ef.Name)
		}

		return i.Suspend(false, false)
	default:
		// TODO: support ublk frontend suspend
		return errors.Wrapf(ErrEngineFrontendLifecycleUnimplemented, "suspend frontend %s is unimplemented", ef.Frontend)
	}
}

// ResumeFrontend resumes the engine frontend. IO operations will be resumed.
func (ef *EngineFrontend) Resume(_ *spdkclient.Client) (err error) {
	ef.Lock()
	defer func() {
		if err != nil {
			if ef.State != types.InstanceStateError {
				ef.log.WithError(err).Warn("Failed to resume engine frontend")
				// Engine is still alive and running and is not really in error state.
				// longhorn-manager will retry to resume the engine.
			}
			ef.ErrorMsg = err.Error()
		} else {
			ef.State = types.InstanceStateRunning
			ef.ErrorMsg = ""

			ef.log.Infof("Resumed engine frontend")
		}

		ef.Unlock()

		ef.UpdateCh <- nil
	}()

	if ef.State == types.InstanceStateRunning {
		return nil
	}
	if ef.isSwitchingOver {
		return errors.Wrapf(ErrSwitchOverTargetPrecondition, "engine frontend %s is switching over target", ef.Name)
	}

	switch ef.Frontend {
	case types.FrontendSPDKTCPBlockdev:
		nvmeTCPInfo := &initiator.NVMeTCPInfo{
			SubsystemNQN: ef.NvmeTcpFrontend.Nqn,
		}
		i, err := initiator.NewInitiator(ef.VolumeName, initiator.HostProc, nvmeTCPInfo, nil)
		if err != nil {
			return errors.Wrapf(err, "failed to create initiator for resuming engine %s", ef.Name)
		}

		ef.log.Info("Resuming engine frontend")
		return i.Resume()
	default:
		// TODO: support ublk frontend resume
		return errors.Wrapf(ErrEngineFrontendLifecycleUnimplemented, "resume frontend %s is unimplemented", ef.Frontend)
	}
}

// SwitchOverTarget switches the backend target for an existing engine frontend.
// For blockdev frontend, the caller must suspend the frontend before switch-over.
// If newEngineName is empty, the function will try to resolve it via targetAddress.
func (ef *EngineFrontend) SwitchOverTarget(spdkClient *spdkclient.Client, newEngineName, targetAddress string) (err error) {
	if targetAddress == "" {
		return errors.Wrapf(ErrSwitchOverTargetInvalidInput, "target address is required for engine frontend %s switchover", ef.Name)
	}

	targetIP, targetPort, err := splitHostPort(targetAddress)
	if err != nil {
		return errors.Wrapf(ErrSwitchOverTargetInvalidInput, "failed to split target address %v for engine frontend %s switchover: %v", targetAddress, ef.Name, err)
	}
	if targetIP == "" || targetPort == 0 {
		return errors.Wrapf(ErrSwitchOverTargetInvalidInput, "invalid target address %q for engine frontend %s switchover", targetAddress, ef.Name)
	}

	updateRequired := false

	ef.Lock()
	if ef.isCreating {
		ef.Unlock()
		return fmt.Errorf("engine frontend %s is still creating", ef.Name)
	}
	if ef.isSwitchingOver {
		ef.Unlock()
		return errors.Wrapf(ErrSwitchOverTargetPrecondition, "engine frontend %s target switchover is already in progress", ef.Name)
	}
	if ef.isExpanding {
		ef.Unlock()
		return errors.Wrapf(ErrSwitchOverTargetPrecondition, "engine frontend %s expansion is in progress", ef.Name)
	}
	if ef.IsRestoring {
		ef.Unlock()
		return errors.Wrapf(ErrSwitchOverTargetPrecondition, "engine frontend %s restore is in progress", ef.Name)
	}
	if ef.NvmeTcpFrontend == nil {
		ef.Unlock()
		return errors.Wrapf(ErrSwitchOverTargetPrecondition, "invalid NvmeTcpFrontend for engine frontend %s switchover", ef.Name)
	}

	oldEngineIP := ef.EngineIP
	oldEngineName := ef.EngineName
	oldTargetIP := ef.NvmeTcpFrontend.TargetIP
	oldTargetPort := ef.NvmeTcpFrontend.TargetPort
	oldNQN := ef.NvmeTcpFrontend.Nqn
	oldNGUID := ef.NvmeTcpFrontend.Nguid
	oldEndpoint := ef.Endpoint
	oldDMDeviceIsBusy := ef.dmDeviceIsBusy
	frontend := ef.Frontend

	resolvedEngineName := newEngineName
	if resolvedEngineName == "" && oldTargetIP == targetIP && oldTargetPort == targetPort {
		// Treat duplicate request to current target as no-op without remote lookup.
		resolvedEngineName = oldEngineName
	}
	if oldTargetIP == targetIP && oldTargetPort == targetPort && oldEngineIP == targetIP && oldEngineName == resolvedEngineName {
		if ef.State != types.InstanceStateError {
			ef.ErrorMsg = ""
		}
		ef.Unlock()
		return nil
	}

	if frontend == types.FrontendSPDKTCPBlockdev && ef.State != types.InstanceStateSuspended {
		state := ef.State
		ef.Unlock()
		return errors.Wrapf(ErrSwitchOverTargetPrecondition, "invalid state %v for engine frontend %s target switchover, must be suspended", state, ef.Name)
	}
	initiatorCreationRequired := frontend == types.FrontendSPDKTCPBlockdev && ef.initiator == nil
	ef.isSwitchingOver = true
	ef.Unlock()

	defer func() {
		ef.Lock()
		ef.isSwitchingOver = false
		ef.Unlock()

		if updateRequired {
			ef.UpdateCh <- nil
		}
	}()

	if resolvedEngineName == "" {
		resolvedEngineName, err = ef.resolveEngineNameByTargetAddress(targetAddress)
		if err != nil {
			return errors.Wrapf(err, "failed to resolve engine name for target address %s", targetAddress)
		}
	}
	newNQN := helpertypes.GetNQN(resolvedEngineName)
	newNGUID := generateNGUID(resolvedEngineName)

	switch frontend {
	case types.FrontendSPDKTCPNvmf:
		ef.Lock()
		ef.EngineIP = targetIP
		ef.EngineName = resolvedEngineName
		ef.NvmeTcpFrontend.TargetIP = targetIP
		ef.NvmeTcpFrontend.TargetPort = targetPort
		ef.NvmeTcpFrontend.Nqn = newNQN
		ef.NvmeTcpFrontend.Nguid = newNGUID
		ef.Endpoint = GetNvmfEndpoint(newNQN, targetIP, targetPort)
		if ef.State != types.InstanceStateError {
			ef.ErrorMsg = ""
		}
		ef.Unlock()
		updateRequired = true

		ef.log.WithFields(logrus.Fields{
			"oldEngineName": oldEngineName,
			"engineName":    resolvedEngineName,
			"oldTargetIP":   oldTargetIP,
			"oldTargetPort": oldTargetPort,
			"targetIP":      targetIP,
			"targetPort":    targetPort,
		}).Info("Switched over engine frontend target")

		// Persist updated record AFTER successful switchover.
		ef.RLock()
		if err := saveEngineFrontendRecord(ef.metadataDir, ef); err != nil {
			ef.log.WithError(err).Warn("Failed to persist engine frontend record after switchover")
		}
		ef.RUnlock()

		return nil

	case types.FrontendSPDKTCPBlockdev:
		if initiatorCreationRequired {
			// Recreate initiator if the cached one is missing but frontend metadata is still valid.
			i, nqn, nguid, initErr := ef.newNvmeTcpInitiator()
			if initErr != nil {
				return errors.Wrapf(ErrSwitchOverTargetInternal, "failed to create initiator for engine frontend %s switchover: %v", ef.Name, initErr)
			}
			ef.Lock()
			ef.initiator = i
			ef.NvmeTcpFrontend.Nqn = nqn
			ef.NvmeTcpFrontend.Nguid = nguid
			ef.Unlock()
		}
		// Do NOT overwrite SubsystemNQN before startNvmeTCPInitiator.
		// The stop path inside startNvmeTCPInitiator uses SubsystemNQN to
		// disconnect the old NVMe controller. If we set newNQN here, the old
		// controller (with oldNQN) would never be disconnected, causing ~30s
		// of kernel NVMe reconnect retries until timeout.
		// startNvmeTCPInitiator will set the correct NQN after connecting
		// the new target via discoverAndConnectNVMeTCPTarget.

		dmDeviceIsBusy, switchErr := ef.startNvmeTCPInitiator(targetIP, targetPort, true, true)
		if switchErr != nil {
			switchErr = errors.Wrapf(ErrSwitchOverTargetInternal, "failed to switch engine frontend %s target to %s: %v", ef.Name, targetAddress, switchErr)

			var rollbackErr error
			if oldTargetIP != "" && oldTargetPort != 0 {
				ef.log.WithError(switchErr).Warnf("Failed to switch target, initiating rollback to previous target %s:%d", oldTargetIP, oldTargetPort)
				if ef.initiator.NVMeTCPInfo != nil {
					ef.initiator.NVMeTCPInfo.SubsystemNQN = oldNQN
				}
				var rollbackDMDeviceIsBusy bool
				if rollbackDMDeviceIsBusy, rollbackErr = ef.startNvmeTCPInitiator(oldTargetIP, oldTargetPort, true, true); rollbackErr != nil {
					rollbackErr = errors.Wrapf(ErrSwitchOverTargetInternal, "failed to rollback engine frontend %s target to %s:%d: %v", ef.Name, oldTargetIP, oldTargetPort, rollbackErr)
					ef.log.WithError(rollbackErr).Errorf("Failed to rollback engine frontend %s target to previous target", ef.Name)
				} else {
					ef.log.Info("Successfully rolled back engine frontend target")
					oldDMDeviceIsBusy = rollbackDMDeviceIsBusy
				}
			}

			// Restore all metadata to original state regardless of rollback success to avoid go struct inconsistency
			ef.Lock()
			ef.EngineIP = oldEngineIP
			ef.EngineName = oldEngineName
			ef.NvmeTcpFrontend.TargetIP = oldTargetIP
			ef.NvmeTcpFrontend.TargetPort = oldTargetPort
			ef.NvmeTcpFrontend.Nqn = oldNQN
			ef.NvmeTcpFrontend.Nguid = oldNGUID
			ef.Endpoint = oldEndpoint
			ef.dmDeviceIsBusy = oldDMDeviceIsBusy

			if rollbackErr != nil {
				combinedErr := multierr.Append(switchErr, rollbackErr)
				ef.State = types.InstanceStateError
				ef.ErrorMsg = combinedErr.Error()
				ef.Unlock()
				updateRequired = true
				return combinedErr
			}
			ef.ErrorMsg = switchErr.Error()
			ef.Unlock()
			updateRequired = true
			return switchErr
		}

		ef.Lock()
		ef.EngineIP = targetIP
		ef.EngineName = resolvedEngineName
		ef.NvmeTcpFrontend.TargetIP = targetIP
		ef.NvmeTcpFrontend.TargetPort = targetPort
		ef.NvmeTcpFrontend.Nqn = newNQN
		ef.NvmeTcpFrontend.Nguid = newNGUID
		ef.Endpoint = ef.getInitiatorEndpoint()
		ef.dmDeviceIsBusy = dmDeviceIsBusy
		if ef.State != types.InstanceStateError {
			ef.ErrorMsg = ""
		}
		ef.Unlock()
		updateRequired = true

		ef.log.WithFields(logrus.Fields{
			"oldEngineName": oldEngineName,
			"engineName":    resolvedEngineName,
			"oldTargetIP":   oldTargetIP,
			"oldTargetPort": oldTargetPort,
			"targetIP":      targetIP,
			"targetPort":    targetPort,
		}).Info("Switched over engine frontend target")

		// Persist updated record AFTER successful switchover.
		ef.RLock()
		if err := saveEngineFrontendRecord(ef.metadataDir, ef); err != nil {
			ef.log.WithError(err).Warn("Failed to persist engine frontend record after switchover")
		}
		ef.RUnlock()

		return nil

	default:
		return errors.Wrapf(ErrSwitchOverTargetPrecondition, "frontend %s does not support target switchover for engine frontend %s", ef.Frontend, ef.Name)
	}
}

func (ef *EngineFrontend) startNvmeTCPInitiator(transportAddress string, transportPort int32, dmDeviceAndEndpointCleanupRequired bool, stop bool) (bool, error) {
	transportServiceID := strconv.Itoa(int(transportPort))
	if ef.startNvmeTCPInitiatorFn != nil {
		return ef.startNvmeTCPInitiatorFn(transportAddress, transportServiceID, dmDeviceAndEndpointCleanupRequired, stop)
	}
	if ef.initiator == nil {
		return false, errors.Wrapf(ErrSwitchOverTargetInternal, "initiator is nil for engine frontend %s", ef.Name)
	}
	return ef.initiator.StartNvmeTCPInitiator(transportAddress, transportServiceID, dmDeviceAndEndpointCleanupRequired, stop)
}

func (ef *EngineFrontend) getInitiatorEndpoint() string {
	if ef.getInitiatorEndpointFn != nil {
		return ef.getInitiatorEndpointFn()
	}
	if ef.initiator == nil {
		return ""
	}
	return ef.initiator.GetEndpoint()
}

func (ef *EngineFrontend) resolveEngineNameByTargetAddress(targetAddress string) (string, error) {
	if ef.resolveEngineNameByTargetAddressFn != nil {
		return ef.resolveEngineNameByTargetAddressFn(targetAddress)
	}

	targetIP, targetPort, err := splitHostPort(targetAddress)
	if err != nil {
		return "", errors.Wrapf(ErrSwitchOverTargetInvalidInput, "failed to split target address %s: %v", targetAddress, err)
	}
	if targetIP == "" || targetPort == 0 {
		return "", errors.Wrapf(ErrSwitchOverTargetInvalidInput, "invalid target address %s", targetAddress)
	}

	targetSpdkClient, err := GetServiceClient(targetAddress)
	if err != nil {
		return "", errors.Wrapf(ErrSwitchOverTargetInternal, "failed to get SPDK client for target address %s: %v", targetAddress, err)
	}
	defer func() {
		if errClose := targetSpdkClient.Close(); errClose != nil {
			ef.log.WithError(errClose).Error("Failed to close target SPDK client")
		}
	}()

	engineMap, err := targetSpdkClient.EngineList()
	if err != nil {
		return "", errors.Wrapf(ErrSwitchOverTargetInternal, "failed to list engines from target address %s: %v", targetAddress, err)
	}

	matchedEngineName := ""
	for engineName, e := range engineMap {
		if e == nil {
			continue
		}
		if e.IP == targetIP && e.Port == targetPort {
			if matchedEngineName != "" {
				return "", errors.Wrapf(ErrSwitchOverTargetPrecondition, "multiple engines match target address %s", targetAddress)
			}
			matchedEngineName = engineName
		}
	}

	if matchedEngineName == "" {
		return "", errors.Wrapf(ErrSwitchOverTargetEngineNotFound, "cannot find engine by target address %s", targetAddress)
	}

	return matchedEngineName, nil
}

func (ef *EngineFrontend) SnapshotCreate(inputSnapshotName string) (snapshotName string, err error) {
	ef.log.Infof("Creating snapshot %s", inputSnapshotName)

	return ef.snapshotOperation(inputSnapshotName, SnapshotOperationCreate, nil)
}

func (ef *EngineFrontend) SnapshotDelete(snapshotName string) (err error) {
	ef.log.Infof("Deleting snapshot %s", snapshotName)

	_, err = ef.snapshotOperation(snapshotName, SnapshotOperationDelete, nil)
	return err
}

func (ef *EngineFrontend) SnapshotRevert(snapshotName string) (err error) {
	ef.log.Infof("Reverting snapshot %s", snapshotName)

	_, err = ef.snapshotOperation(snapshotName, SnapshotOperationRevert, nil)
	return err
}

func (ef *EngineFrontend) SnapshotPurge() (err error) {
	ef.log.Infof("Purging snapshots")

	_, err = ef.snapshotOperation("", SnapshotOperationPurge, nil)
	return err
}

func (ef *EngineFrontend) snapshotOperation(inputSnapshotName string, snapshotOp SnapshotOperationType, opts any) (snapshotName string, err error) {
	updateRequired := false

	ef.Lock()
	if ef.isCreating {
		ef.Unlock()
		return "", fmt.Errorf("engine frontend %s is still creating", ef.Name)
	}
	if ef.isSwitchingOver {
		ef.Unlock()
		return "", errors.Wrapf(ErrSwitchOverTargetPrecondition, "engine frontend %s is switching over target", ef.Name)
	}

	defer func() {
		ef.Unlock()

		if updateRequired {
			ef.UpdateCh <- nil
		}
	}()

	var engineFrontendErr error

	defer func() {
		if engineFrontendErr != nil {
			if ef.State != types.InstanceStateError {
				ef.log.Error("Setting engine frontend to error state due to snapshot operation failure")
				ef.State = types.InstanceStateError
				updateRequired = true
			}
			ef.ErrorMsg = engineFrontendErr.Error()
		} else {
			if ef.State != types.InstanceStateError {
				ef.ErrorMsg = ""
			}
		}
	}()

	// Pause the IO, flush outstanding IO and attempt to synchronize filesystem by suspending the NVMe initiator
	if snapshotOp == SnapshotOperationCreate {
		if ef.isSuspendSupported() {
			ef.log.Infof("Suspending before the snapshot operation %s for snapshot %s", snapshotOp, inputSnapshotName)
			if err := ef.suspend(false, false); err != nil {
				return "", errors.Wrapf(err, "failed to suspend before the snapshot operation %s for snapshot %q", snapshotOp, inputSnapshotName)
			}
			defer func() {
				ef.log.Infof("Resuming after the snapshot operation %s for snapshot %s", snapshotOp, inputSnapshotName)
				if resumeErr := ef.resume(); resumeErr != nil {
					engineFrontendErr = errors.Wrapf(resumeErr, "failed to resume after the snapshot operation %s for snapshot %q", snapshotOp, inputSnapshotName)
					return
				}
				ef.log.Infof("Resumed after the snapshot operation %s for snapshot %q", snapshotOp, inputSnapshotName)
			}()
		}
	}

	engineSpdkClient, err := GetServiceClient(net.JoinHostPort(ef.EngineIP, strconv.Itoa(types.SPDKServicePort)))
	if err != nil {
		return "", errors.Wrapf(err, "failed to get SPDK client to perform snapshot operation %s for snapshot %q", snapshotOp, inputSnapshotName)
	}
	defer func() {
		if errClose := engineSpdkClient.Close(); errClose != nil {
			ef.log.WithError(errClose).Error("Failed to close SPDK client")
		}
	}()

	switch snapshotOp {
	case SnapshotOperationCreate:
		return engineSpdkClient.EngineSnapshotCreate(ef.EngineName, inputSnapshotName)
	case SnapshotOperationDelete:
		return "", engineSpdkClient.EngineSnapshotDelete(ef.EngineName, inputSnapshotName)
	case SnapshotOperationRevert:
		return "", engineSpdkClient.EngineSnapshotRevert(ef.EngineName, inputSnapshotName)
	case SnapshotOperationPurge:
		return "", engineSpdkClient.EngineSnapshotPurge(ef.EngineName)
	default:
		return "", fmt.Errorf("unknown snapshot operation %v for snapshot %q", snapshotOp, inputSnapshotName)
	}
}

func (ef *EngineFrontend) isSuspendSupported() bool {
	return ef.Frontend == types.FrontendSPDKTCPBlockdev && ef.Endpoint != ""
}

func (ef *EngineFrontend) suspend(noflush, nolockfs bool) error {
	if ef.initiator == nil {
		return fmt.Errorf("initiator is not initialized for engine frontend %s", ef.Name)
	}
	return ef.initiator.Suspend(noflush, nolockfs)
}

func (ef *EngineFrontend) resume() error {
	if ef.initiator == nil {
		return fmt.Errorf("initiator is not initialized for engine frontend %s", ef.Name)
	}
	return ef.initiator.Resume()
}

// ValidateAndUpdate validates the engine frontend (initiator-side) state and updates
// fields (e.g., Endpoint) as needed. Called periodically by the server verify loop.
// This only validates the local initiator/device state — target-side subsystem
// validation is the responsibility of the Engine.
func (ef *EngineFrontend) ValidateAndUpdate(spdkClient *spdkclient.Client) (err error) {
	updateRequired := false

	ef.Lock()
	defer func() {
		if err != nil {
			if ef.State != types.InstanceStateError {
				ef.log.WithError(err).Error("Setting engine frontend to error state due to validation failure")
				ef.State = types.InstanceStateError
				updateRequired = true
			}
			ef.ErrorMsg = err.Error()
		}
		ef.Unlock()

		if updateRequired {
			ef.UpdateCh <- nil
		}
	}()

	if ef.State != types.InstanceStateRunning {
		return nil
	}

	if ef.isCreating {
		ef.log.Debug("Engine frontend is creating, will skip the validation and update")
		return nil
	}

	if ef.isSwitchingOver {
		ef.log.Debug("Engine frontend is switching over target, will skip the validation and update")
		return nil
	}

	if ef.isExpanding {
		ef.log.Debug("Engine frontend is expanding, will skip the validation and update")
		return nil
	}

	if ef.IsRestoring {
		ef.log.Debug("Engine frontend is restoring, will skip the validation and update")
		return nil
	}

	return ef.validateAndUpdateFrontend(spdkClient)
}

func (ef *EngineFrontend) validateAndUpdateFrontend(client *spdkclient.Client) (err error) {
	if !types.IsFrontendSupported(ef.Frontend) {
		return fmt.Errorf("unknown frontend type %s", ef.Frontend)
	}
	if ef.Frontend == types.FrontendEmpty {
		if ef.Endpoint != "" {
			return fmt.Errorf("found non-empty endpoint %s for engine frontend %s with empty frontend", ef.Endpoint, ef.Name)
		}
		return nil
	}
	switch ef.Frontend {
	case types.FrontendUBLK:
		return ef.validateAndUpdateUblkFrontend(client)
	case types.FrontendSPDKTCPBlockdev, types.FrontendSPDKTCPNvmf:
		return ef.validateAndUpdateNvmeTcpFrontend()
	default:
		return fmt.Errorf("unsupported frontend type %s for engine frontend %s validation", ef.Frontend, ef.Name)
	}
}

func (ef *EngineFrontend) validateAndUpdateUblkFrontend(client *spdkclient.Client) (err error) {
	defer func() {
		if err != nil {
			err = errors.Wrapf(err, "failed to validateAndUpdateUblkFrontend for engine frontend %v", ef.Name)
		}
	}()
	if ef.UblkFrontend == nil {
		return fmt.Errorf("UblkFrontend is nil")
	}

	ublkDeviceList, err := client.UblkGetDisks(ef.UblkFrontend.UblkID)
	if err != nil {
		return err
	}
	for _, ublkDevice := range ublkDeviceList {
		if ublkDevice.BdevName == ef.EngineName && ublkDevice.ID != ef.UblkFrontend.UblkID {
			return fmt.Errorf("found mismatching between UblkFrontend.UblkID %v and actual ublk device id %v", ef.UblkFrontend.UblkID, ublkDevice.ID)
		}
	}
	return nil
}

// validateAndUpdateNvmeTcpFrontend validates the initiator-side NVMe/TCP state:
// ensures the initiator exists, loads device info, and checks endpoint consistency.
func (ef *EngineFrontend) validateAndUpdateNvmeTcpFrontend() (err error) {
	if ef.NvmeTcpFrontend == nil {
		return fmt.Errorf("failed to validateAndUpdateNvmeTcpFrontend for engine frontend %v: NvmeTcpFrontend is nil", ef.Name)
	}

	switch ef.Frontend {
	case types.FrontendSPDKTCPBlockdev:
		if ef.initiator == nil {
			nvmeTCPInfo := &initiator.NVMeTCPInfo{
				SubsystemNQN: ef.NvmeTcpFrontend.Nqn,
			}
			i, err := initiator.NewInitiator(ef.VolumeName, initiator.HostProc, nvmeTCPInfo, nil)
			if err != nil {
				return errors.Wrapf(err, "failed to create initiator for engine frontend %v during frontend validation and update", ef.Name)
			}
			ef.initiator = i
		}
		if ef.initiator.NVMeTCPInfo == nil {
			return fmt.Errorf("invalid initiator with nil NvmeTcpInfo")
		}
		if err := ef.initiator.LoadNVMeDeviceInfo(ef.initiator.NVMeTCPInfo.TransportAddress, ef.initiator.NVMeTCPInfo.TransportServiceID, ef.initiator.NVMeTCPInfo.SubsystemNQN); err != nil {
			if strings.Contains(err.Error(), "connecting state") ||
				strings.Contains(err.Error(), "resetting state") {
				ef.log.WithError(err).Warn("Ignored to validate and update engine frontend, because the device is still in a transient state")
				return nil
			}
			return err
		}
		if err := ef.initiator.LoadEndpointForNvmeTcpFrontend(ef.dmDeviceIsBusy); err != nil {
			return err
		}
		blockDevEndpoint := ef.initiator.GetEndpoint()
		if ef.Endpoint == "" {
			ef.Endpoint = blockDevEndpoint
		}
		if ef.Endpoint != blockDevEndpoint {
			return fmt.Errorf("found mismatching between engine frontend endpoint %s and actual block device endpoint %s for engine frontend %s", ef.Endpoint, blockDevEndpoint, ef.Name)
		}
	case types.FrontendSPDKTCPNvmf:
		nvmfEndpoint := GetNvmfEndpoint(ef.NvmeTcpFrontend.Nqn, ef.NvmeTcpFrontend.TargetIP, ef.NvmeTcpFrontend.TargetPort)
		if ef.Endpoint == "" {
			ef.Endpoint = nvmfEndpoint
		}
		if ef.Endpoint != nvmfEndpoint {
			return fmt.Errorf("found mismatching between engine frontend endpoint %s and actual nvmf endpoint %s for engine frontend %s", ef.Endpoint, nvmfEndpoint, ef.Name)
		}
	default:
		return fmt.Errorf("unknown frontend type %s", ef.Frontend)
	}

	return nil
}

// RecoverFromHost attempts to recover the engine frontend's initiator state by
// detecting existing NVMe controllers and dm-devices on the host. This is called
// during server startup for engine frontends that were persisted before restart.
//
// On success, the frontend transitions to Running state.
// On failure, it transitions to Error state so that the upper-layer controller
// can reconcile (e.g. by calling Delete + Create).
func (ef *EngineFrontend) RecoverFromHost(spdkClient *spdkclient.Client) error {
	ef.Lock()
	if ef.State != types.InstanceStatePending {
		ef.Unlock()
		return fmt.Errorf("invalid state %s for engine frontend %s recovery", ef.State, ef.Name)
	}
	ef.Unlock()

	var recoverErr error
	var deviceNotFound bool

	defer func() {
		ef.Lock()
		defer ef.Unlock()

		if deviceNotFound {
			// Device not found on host — record already removed, nothing to reconcile.
			return
		}
		if recoverErr != nil {
			ef.log.WithError(recoverErr).Errorf("Failed to recover engine frontend %s from host", ef.Name)
			ef.State = types.InstanceStateError
			ef.ErrorMsg = recoverErr.Error()
		} else {
			ef.State = types.InstanceStateRunning
			ef.ErrorMsg = ""
			ef.log.Info("Successfully recovered engine frontend from host")
		}
		ef.UpdateCh <- nil
	}()

	switch ef.Frontend {
	case types.FrontendEmpty:
		// No initiator to recover for empty frontend.
		return nil

	case types.FrontendSPDKTCPNvmf:
		// For NVMe-oF (non-blockdev) frontend, there is no local initiator.
		// Just reconstruct the endpoint from persisted TargetPort and EngineIP.
		nqn := helpertypes.GetNQN(ef.EngineName)
		nguid := generateNGUID(ef.EngineName)

		ef.Lock()
		ef.NvmeTcpFrontend.Nqn = nqn
		ef.NvmeTcpFrontend.Nguid = nguid
		if ef.NvmeTcpFrontend.TargetIP == "" {
			ef.NvmeTcpFrontend.TargetIP = ef.EngineIP
		}
		if ef.NvmeTcpFrontend.TargetPort != 0 {
			ef.Endpoint = GetNvmfEndpoint(nqn, ef.NvmeTcpFrontend.TargetIP, ef.NvmeTcpFrontend.TargetPort)
		}
		ef.Unlock()

		return nil

	case types.FrontendSPDKTCPBlockdev:
		// Recover the NVMe-oF initiator (blockdev frontend with dm-device).
		i, nqn, nguid, err := ef.newNvmeTcpInitiator()
		if err != nil {
			recoverErr = errors.Wrapf(err, "failed to create NVMe/TCP initiator for recovery of engine frontend %s", ef.Name)
			return recoverErr
		}

		ef.Lock()
		ef.NvmeTcpFrontend.Nqn = nqn
		ef.NvmeTcpFrontend.Nguid = nguid
		ef.NvmeTcpFrontend.TargetIP = ef.EngineIP
		ef.Unlock()

		// Try to load the existing NVMe device info from sysfs.
		// Use empty transport address/port since we want to discover by NQN.
		if err := i.LoadNVMeDeviceInfo("", "", nqn); err != nil {
			if strings.Contains(err.Error(), helpertypes.ErrorMessageCannotFindValidNvmeDevice) {
				ef.log.WithError(err).Warnf("NVMe device not found on host during recovery of engine frontend %s, removing persisted record", ef.Name)
				if removeErr := removeEngineFrontendRecord(ef.metadataDir, ef.VolumeName); removeErr != nil {
					ef.log.WithError(removeErr).Warn("Failed to remove engine frontend record")
				}
				deviceNotFound = true
				return ErrRecoverDeviceNotFound
			}
			recoverErr = errors.Wrapf(err, "failed to load NVMe device info during recovery of engine frontend %s", ef.Name)
			return recoverErr
		}

		// Try to load the existing dm-device endpoint.
		if err := i.LoadEndpointForNvmeTcpFrontend(false); err != nil {
			recoverErr = errors.Wrapf(err, "failed to load endpoint during recovery of engine frontend %s", ef.Name)
			return recoverErr
		}

		ef.Lock()
		ef.initiator = i
		ef.Endpoint = i.GetEndpoint()
		// Recover target port from the detected transport service ID.
		if transportServiceID := i.GetTransportServiceID(); transportServiceID != "" {
			if port, parseErr := strconv.Atoi(transportServiceID); parseErr == nil {
				ef.NvmeTcpFrontend.TargetPort = int32(port)
			}
		}
		// Recover target IP from the detected transport address.
		if transportAddress := i.GetTransportAddress(); transportAddress != "" {
			ef.NvmeTcpFrontend.TargetIP = transportAddress
			ef.EngineIP = transportAddress
		}
		ef.Unlock()

		return nil

	default:
		recoverErr = fmt.Errorf("unsupported frontend type %s for recovery of engine frontend %s", ef.Frontend, ef.Name)
		return recoverErr
	}
}
</file>

<file path="pkg/spdk/expand_test.go">
package spdk

import (
	"context"
	"fmt"
	"strings"
	"sync"
	"time"

	"github.com/cockroachdb/errors"

	"github.com/longhorn/types/pkg/generated/spdkrpc"

	helpertypes "github.com/longhorn/go-spdk-helper/pkg/types"

	clientpkg "github.com/longhorn/longhorn-spdk-engine/pkg/client"
	lhtypes "github.com/longhorn/longhorn-spdk-engine/pkg/types"

	. "gopkg.in/check.v1"
)

func (s *TestSuite) TestEngineFinishExpansionSuccessClearsErrorState(c *C) {
	fmt.Println("Testing Engine finish expansion success clears error state")

	e := NewEngine("engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))
	e.State = lhtypes.InstanceStateError
	e.ErrorMsg = "previous failure"

	e.finishExpansion(10, 20, nil)

	c.Assert(e.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateRunning))
	c.Assert(e.ErrorMsg, Equals, "")
	c.Assert(e.SpecSize, Equals, uint64(20))
}

func (s *TestSuite) TestEngineFinishExpansionFailureSetsErrorState(c *C) {
	fmt.Println("Testing Engine finish expansion failure sets error state")

	e := NewEngine("engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))
	e.State = lhtypes.InstanceStateRunning
	e.ErrorMsg = ""
	e.SpecSize = 10

	e.finishExpansion(10, 20, errors.New("expand failed"))

	c.Assert(e.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateError))
	c.Assert(e.ErrorMsg, Not(Equals), "")
	c.Assert(e.lastExpansionError, Not(Equals), "")
	c.Assert(e.lastExpansionFailedAt, Not(Equals), "")
	c.Assert(e.SpecSize, Equals, uint64(10))
}

func (s *TestSuite) TestEngineFinishExpansionFailureRestoresOriginalSize(c *C) {
	fmt.Println("Testing Engine finish expansion failure restores original size when spec size was updated during expand")

	e := NewEngine("engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))
	e.SpecSize = 20

	e.finishExpansion(10, 20, errors.New("expand failed"))

	c.Assert(e.SpecSize, Equals, uint64(10))
}

func (s *TestSuite) TestEngineFrontendFinishExpansionSuccessClearsErrorState(c *C) {
	fmt.Println("Testing EngineFrontend finish expansion success clears error state")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, 0, 0, make(chan interface{}, 1))
	ef.State = lhtypes.InstanceStateError
	ef.ErrorMsg = "previous failure"
	ef.lastExpansionError = ""

	ef.finishExpansion(10, true, 20, nil, "", "", 15)

	c.Assert(ef.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateRunning))
	c.Assert(ef.ErrorMsg, Equals, "")
	c.Assert(ef.SpecSize, Equals, uint64(20))
	c.Assert(ef.ActualSize, Equals, uint64(15))
	c.Assert(ef.isExpanding, Equals, false)
}

func (s *TestSuite) TestEngineFrontendFinishExpansionExpandedWithError(c *C) {
	fmt.Println("Testing EngineFrontend finish expansion with error after expansion")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, 0, 0, make(chan interface{}, 1))

	ef.finishExpansion(10, true, 20, errors.New("post expansion frontend failure"), "", "", 15)

	c.Assert(ef.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateError))
	c.Assert(ef.ErrorMsg, Not(Equals), "")
	c.Assert(ef.SpecSize, Equals, uint64(20))
	c.Assert(ef.ActualSize, Equals, uint64(15))
	c.Assert(ef.lastExpansionError, Not(Equals), "")
	c.Assert(ef.isExpanding, Equals, false)
}

func (s *TestSuite) TestEngineFrontendFinishExpansionFailureWithoutExpansion(c *C) {
	fmt.Println("Testing EngineFrontend finish expansion failure without expansion")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, 0, 0, make(chan interface{}, 1))
	ef.SpecSize = 10

	ef.finishExpansion(10, false, 20, errors.New("expand failed before backend expansion"), "", "", 0)

	c.Assert(ef.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateError))
	c.Assert(ef.SpecSize, Equals, uint64(10))
	c.Assert(ef.ActualSize, Equals, uint64(0))
	c.Assert(ef.lastExpansionError, Not(Equals), "")
	c.Assert(ef.isExpanding, Equals, false)
}

func (s *TestSuite) TestEngineFinishExpansionPartialFailureKeepsOriginalSize(c *C) {
	fmt.Println("Testing Engine finish expansion partial failure keeps original size")

	e := NewEngine("engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))
	e.lastExpansionError = "replica expand failed"

	e.finishExpansion(10, 20, nil)

	c.Assert(e.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateRunning))
	c.Assert(e.ErrorMsg, Equals, "")
	c.Assert(e.SpecSize, Equals, uint64(10))
	c.Assert(e.lastExpansionFailedAt, Not(Equals), "")
}

func (s *TestSuite) TestEngineFinishExpansionPartialFailureRestoresOriginalSize(c *C) {
	fmt.Println("Testing Engine finish expansion partial failure restores original size when spec size was updated during expand")

	e := NewEngine("engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))
	e.SpecSize = 20
	e.lastExpansionError = "replica expand failed"

	e.finishExpansion(10, 20, nil)

	c.Assert(e.SpecSize, Equals, uint64(10))
}

func (s *TestSuite) TestEngineFrontendFinishExpansionPartialFailureKeepsOriginalSize(c *C) {
	fmt.Println("Testing EngineFrontend finish expansion partial failure keeps original size")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, 0, 0, make(chan interface{}, 1))

	ef.finishExpansion(10, false, 20, nil, "replica expand failed", "2026-03-10T00:00:00Z", 12)

	c.Assert(ef.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateRunning))
	c.Assert(ef.ErrorMsg, Equals, "")
	c.Assert(ef.SpecSize, Equals, uint64(10))
	c.Assert(ef.ActualSize, Equals, uint64(12))
	c.Assert(ef.lastExpansionError, Equals, "replica expand failed")
	c.Assert(ef.lastExpansionFailedAt, Equals, "2026-03-10T00:00:00Z")
	c.Assert(ef.isExpanding, Equals, false)
}

func (s *TestSuite) TestEngineFrontendRequireExpansionGuards(c *C) {
	fmt.Println("Testing EngineFrontend require expansion guards")

	efInProgress := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, 0, 0, make(chan interface{}, 1))
	efInProgress.isExpanding = true
	_, err := efInProgress.requireExpansion(context.Background(), nil, 20)
	c.Assert(errors.Is(err, ErrExpansionInProgress), Equals, true)

	efRestoring := NewEngineFrontend("ef-b", "engine-b", "vol-b", lhtypes.FrontendSPDKTCPBlockdev, 10, 0, 0, make(chan interface{}, 1))
	efRestoring.IsRestoring = true
	_, err = efRestoring.requireExpansion(context.Background(), nil, 20)
	c.Assert(errors.Is(err, ErrRestoringInProgress), Equals, true)

	efSmaller := NewEngineFrontend("ef-c", "engine-c", "vol-c", lhtypes.FrontendSPDKTCPBlockdev, 20, 0, 0, make(chan interface{}, 1))
	_, err = efSmaller.requireExpansion(context.Background(), nil, 10)
	c.Assert(errors.Is(err, ErrExpansionInvalidSize), Equals, true)

	efUnaligned := NewEngineFrontend("ef-d", "engine-d", "vol-d", lhtypes.FrontendSPDKTCPBlockdev, 10*helpertypes.MiB, 0, 0, make(chan interface{}, 1))
	notAlignedSize := uint64((11 * helpertypes.MiB) + 1)
	_, err = efUnaligned.requireExpansion(context.Background(), nil, notAlignedSize)
	c.Assert(errors.Is(err, ErrExpansionInvalidSize), Equals, true)
}

func (s *TestSuite) TestEngineExpandPrecheckGuards(c *C) {
	fmt.Println("Testing Engine expand precheck guards")

	eInProgress := NewEngine("engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))
	eInProgress.isExpanding = true
	_, err := eInProgress.ExpandPrecheck(nil, 20)
	c.Assert(errors.Is(err, ErrExpansionInProgress), Equals, true)

	eRestoring := NewEngine("engine-b", "vol-b", lhtypes.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))
	eRestoring.IsRestoring = true
	_, err = eRestoring.ExpandPrecheck(nil, 20)
	c.Assert(errors.Is(err, ErrRestoringInProgress), Equals, true)
}

func (s *TestSuite) TestHandleReplicaExpandResult(c *C) {
	fmt.Println("Testing Engine handle replica expand result")

	eAllFailed := NewEngine("engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))
	replicaClientsAllFailed := map[string]*clientpkg.SPDKClient{
		"r1": nil,
		"r2": nil,
	}
	failedAll := map[string]error{
		"r1": errors.New("x"),
		"r2": errors.New("y"),
	}
	err := eAllFailed.handleReplicaExpandResult(replicaClientsAllFailed, failedAll)
	c.Assert(err, NotNil)
	c.Assert(strings.Contains(err.Error(), "all replicas failed to expand"), Equals, true)

	ePartial := NewEngine("engine-b", "vol-b", lhtypes.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))
	ePartial.ReplicaStatusMap = map[string]*EngineReplicaStatus{
		"r1": &EngineReplicaStatus{Mode: lhtypes.ModeRW},
		"r2": &EngineReplicaStatus{Mode: lhtypes.ModeRW},
	}
	replicaClientsPartial := map[string]*clientpkg.SPDKClient{
		"r1": nil,
		"r2": nil,
	}
	failedPartial := map[string]error{
		"r1": errors.New("x"),
	}
	err = ePartial.handleReplicaExpandResult(replicaClientsPartial, failedPartial)
	c.Assert(err, IsNil)
	c.Assert(ePartial.ReplicaStatusMap["r1"].Mode, Equals, lhtypes.Mode(lhtypes.ModeERR))
	c.Assert(ePartial.ReplicaStatusMap["r2"].Mode, Equals, lhtypes.Mode(lhtypes.ModeRW))
	c.Assert(ePartial.lastExpansionError, Not(Equals), "")
}

func (s *TestSuite) TestExpandDoesNotBlockConcurrentGet(c *C) {
	fmt.Println("Testing Expand lock scope does not block concurrent Get calls")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1))
	ef.State = lhtypes.InstanceStateRunning
	ef.EngineIP = "10.0.0.1"

	// Simulate the Phase 1 lock pattern from Expand: acquire, read, release
	var wg sync.WaitGroup
	getCh := make(chan *spdkrpc.EngineFrontend, 1)
	phase2ReadyCh := make(chan struct{})
	allowPhase3Ch := make(chan struct{})

	// Start a goroutine that simulates Phase 1 of Expand (lock → read → unlock)
	// then holds a "long operation" before Phase 3 (re-lock → finishExpansion → unlock)
	wg.Add(1)
	go func() {
		defer wg.Done()

		// Phase 1: lock, read, unlock
		ef.Lock()
		_ = ef.SpecSize
		ef.Unlock()
		close(phase2ReadyCh)

		// Simulate Phase 2: long operation without lock
		<-allowPhase3Ch

		// Phase 3: re-lock, update state, unlock
		ef.Lock()
		ef.SpecSize = 2048
		ef.Unlock()
	}()

	// Wait until the goroutine enters Phase 2 (unlocked state)
	select {
	case <-phase2ReadyCh:
	case <-time.After(2 * time.Second):
		c.Fatal("Expand simulation did not reach Phase 2 in time")
	}

	// Concurrent Get() should not block during Phase 2 (write lock is released)
	wg.Add(1)
	go func() {
		defer wg.Done()
		result := ef.Get()
		getCh <- result
	}()

	select {
	case result := <-getCh:
		c.Assert(result, NotNil)
		c.Assert(result.Name, Equals, "ef-a")
	case <-time.After(2 * time.Second):
		c.Fatal("Get() call was blocked — potential deadlock due to Expand holding lock during long operations")
	}

	// Get() completed; now allow Phase 3 to proceed and verify state update.
	close(allowPhase3Ch)
	wg.Wait()
	c.Assert(ef.SpecSize, Equals, uint64(2048))
}

func (s *TestSuite) TestExpandCapturesTargetAddressWithNilNvmeTcpFrontend(c *C) {
	fmt.Println("Testing Expand safely computes targetAddress when NvmeTcpFrontend is nil")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendUBLK, 1024, 0, 0, make(chan interface{}, 1))
	ef.NvmeTcpFrontend = nil // force nil, simulating UBLK-only scenario

	// Test the targetAddress capture logic
	ef.Lock()
	var targetAddress string
	if ef.NvmeTcpFrontend != nil {
		targetAddress = "should-not-be-set"
	}
	ef.Unlock()

	c.Assert(targetAddress, Equals, "")
}

func (s *TestSuite) TestExpandResumeGuardsNilInitiator(c *C) {
	fmt.Println("Testing Expand resume defer guards against nil initiator")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1))
	ef.initiator = nil

	// Simulate the resume defer guard — should NOT panic.
	c.Assert(func() {
		if ef.initiator != nil {
			_ = ef.initiator.Resume()
		}
	}, Not(PanicMatches), ".*")
}
</file>

<file path="pkg/spdk/log.go">
package spdk

import (
	"strings"

	"github.com/sirupsen/logrus"

	spdkclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
)

func svcLogSetLevel(spdkClient *spdkclient.Client, level string) error {
	log := logrus.WithFields(logrus.Fields{
		"level": level,
	})

	log.Trace("Setting log level")

	if _, err := spdkClient.LogSetLevel(level); err != nil {
		return err
	}
	if _, err := spdkClient.LogSetPrintLevel(level); err != nil {
		return err
	}
	return nil

}

func svcLogSetFlags(spdkClient *spdkclient.Client, flags string) (err error) {
	log := logrus.WithFields(logrus.Fields{
		"flags": flags,
	})

	log.Trace("Setting log flags")

	if flags == "" {
		_, err = spdkClient.LogClearFlag("all")
		return err
	}

	flagMap := commaSeparatedStringToMap(flags)
	if _, ok := flagMap["all"]; ok {
		_, err = spdkClient.LogSetFlag(flags)
		return err
	}

	currentFlagMap, err := spdkClient.LogGetFlags()
	if err != nil {
		return err
	}

	for flag, enabled := range currentFlagMap {
		targetFlagEnabled := flagMap[flag]
		if enabled != targetFlagEnabled {
			if targetFlagEnabled {
				_, err = spdkClient.LogSetFlag(flag)
			} else {
				_, err = spdkClient.LogClearFlag(flag)
			}
			if err != nil {
				return err
			}
		}
	}

	return nil
}

func commaSeparatedStringToMap(flags string) map[string]bool {
	flagMap := make(map[string]bool)
	if flags == "" {
		return flagMap
	}

	flagList := strings.Split(flags, ",")
	for _, flag := range flagList {
		flagMap[flag] = true
	}

	return flagMap
}

func svcLogGetLevel(spdkClient *spdkclient.Client) (level string, err error) {
	logrus.Trace("Getting log level")

	return spdkClient.LogGetPrintLevel()
}

func svcLogGetFlags(spdkClient *spdkclient.Client) (flags string, err error) {
	logrus.Trace("Getting log flags")

	var flagsMap map[string]bool

	flagsMap, err = spdkClient.LogGetFlags()
	if err != nil {
		return "", err
	}
	return mapToCommaSeparatedString(flagsMap), nil
}

func mapToCommaSeparatedString(m map[string]bool) string {
	if len(m) == 0 {
		return ""
	}

	var s string
	for k, v := range m {
		if v {
			s += k + ","
		}
	}
	return s[:len(s)-1]
}
</file>

<file path="pkg/spdk/replica.go">
package spdk

import (
	"context"
	"fmt"
	"math"
	"net"
	"slices"
	"strconv"
	"strings"
	"sync"
	"time"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"go.uber.org/multierr"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/backupstore"
	"github.com/longhorn/go-spdk-helper/pkg/jsonrpc"
	"github.com/longhorn/types/pkg/generated/spdkrpc"

	btypes "github.com/longhorn/backupstore/types"
	butil "github.com/longhorn/backupstore/util"
	commonbitmap "github.com/longhorn/go-common-libs/bitmap"
	commonnet "github.com/longhorn/go-common-libs/net"
	spdkclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
	spdktypes "github.com/longhorn/go-spdk-helper/pkg/spdk/types"
	helpertypes "github.com/longhorn/go-spdk-helper/pkg/types"
	helperutil "github.com/longhorn/go-spdk-helper/pkg/util"

	"github.com/longhorn/longhorn-spdk-engine/pkg/api"
	"github.com/longhorn/longhorn-spdk-engine/pkg/client"
	"github.com/longhorn/longhorn-spdk-engine/pkg/types"
	"github.com/longhorn/longhorn-spdk-engine/pkg/util"

	safelog "github.com/longhorn/longhorn-spdk-engine/pkg/log"
)

const (
	restorePeriodicRefreshInterval = 2 * time.Second

	lvolRangeShallowCopyLength = uint64(1 << 8)
)

type Replica struct {
	sync.RWMutex

	ctx context.Context

	// Head should be the only writable lvol in the regular Replica lvol chain/map.
	// And it is the last entry of ActiveChain if it is not nil.
	Head *Lvol
	// ActiveChain stores the backing image info in index 0.
	// If a replica does not contain a backing image, the first entry will be nil.
	// The last entry of the chain should be the head lvol if it exists.
	ActiveChain []*Lvol
	// SnapshotLvolMap map[<snapshot lvol name>]. <snapshot lvol name> consists of `<replica name>-snap-<snapshot name>`
	SnapshotLvolMap map[string]*Lvol
	BackingImage    *Lvol

	Name    string
	Alias   string
	LvsName string
	LvsUUID string
	Nqn     string

	SpecSize   uint64
	ActualSize uint64

	IP        string
	PortStart int32
	PortEnd   int32

	State    types.InstanceState
	ErrorMsg string

	IsExposed               bool
	SnapshotChecksumEnabled bool

	// SnapshotLvolHashStatusMap map[<snapshot lvol name>]LvolHashStatus.
	SnapshotLvolHashStatusMap sync.Map

	// reconstructRequired will be set to true when stopping an errored replica
	reconstructRequired bool

	// The rebuilding destination replica should cache this info
	isRebuilding       bool
	rebuildingDstCache RebuildingDstCache
	lastRebuildingAt   time.Time

	// QoS limit in MB/s for rebuilding operations
	rebuildingQosLimitMbps int64

	// The rebuilding source replica should cache this info
	rebuildingSrcCache RebuildingSrcCache

	// The cloning destination replica should cache this info
	isSnapshotCloning       bool
	snapshotCloningDstCache SnapshotCloningDstCache

	// The cloning source replica should cache this info
	snapshotCloningSrcCache map[string]*SnapshotCloningSrcCache

	isRestoring bool
	restore     *Restore

	portAllocator *commonbitmap.Bitmap
	// UpdateCh should not be protected by the replica lock
	UpdateCh chan interface{}

	log *safelog.SafeLogger

	// TODO: Record error message
}

type LvolHashStatus struct {
	State            string
	Error            string
	Checksum         string
	PreviousChecksum string
}

type RebuildingDstCache struct {
	rebuildingLvol        *Lvol
	rebuildingPort        int32
	rebuildingLvolAddress string

	srcReplicaName           string
	srcReplicaAddress        string
	externalSnapshotName     string
	externalSnapshotBdevName string

	// rebuildingSnapshotMap is map[<snapshot name>]
	rebuildingSnapshotMap map[string]*api.Lvol
	rebuildingSize        uint64
	rebuildingError       string
	rebuildingState       string

	processedSnapshotList  []string
	processedSnapshotsSize uint64

	processingSnapshotName      string
	processingState             string
	processingSize              uint64
	snapshotTotalRebuildingSize uint64
}

type RebuildingSrcCache struct {
	dstReplicaName string
	// dstRebuildingBdev is the result of attaching the rebuilding lvol exposed by the dst replica
	dstRebuildingBdevName string

	exposedSnapshotAlias string
	exposedSnapshotPort  int32

	shallowCopySnapshotName string
	shallowCopyOpID         uint32
	shallowCopyStatus       ShallowCopyStatus
	isRangeShallowCopy      bool
}

type ShallowCopyStatus struct {
	State           string `json:"state"`
	Error           string `json:"error,omitempty"`
	HandledClusters uint64 `json:"handled_clusters"`
	TotalClusters   uint64 `json:"total_clusters"`
	// HandledRangeClusters is the number of clusters all finished range shallow copies handled for this snapshot.
	HandledRangeClusters uint64 `json:"handled_range_clusters"`
	// CurrentRangeState is the state of the current range shallow copy.
	CurrentRangeState string `json:"current_range_state"`
}

type SnapshotCloningDstCache struct {
	snapshotName string

	cloningLvol        *Lvol
	cloningPort        int32
	cloningLvolAddress string

	srcReplicaName    string
	srcReplicaAddress string

	processedClusters uint64
	totalClusters     uint64
	cloningError      string
	cloningState      string
	monitorCancelFunc context.CancelFunc
}

type SnapshotCloningSrcCache struct {
	dstReplicaName string
	// dstCloningBdevName is the result of attaching the cloning lvol exposed by the dst replica
	dstCloningBdevName string

	snapshotName   string
	deepCopyOpID   uint32
	deepCopyStatus DeepCopyStatus
}
type DeepCopyStatus struct {
	State             string `json:"state"`
	ProcessedClusters uint64 `json:"processed_clusters"`
	TotalClusters     uint64 `json:"total_clusters"`
	Error             string `json:"error,omitempty"`
}

func ServiceReplicaToProtoReplica(r *Replica) *spdkrpc.Replica {
	res := &spdkrpc.Replica{
		Name:      r.Name,
		LvsName:   r.LvsName,
		LvsUuid:   r.LvsUUID,
		SpecSize:  r.SpecSize,
		Snapshots: map[string]*spdkrpc.Lvol{},
		Ip:        r.IP,
		PortStart: r.PortStart,
		PortEnd:   r.PortEnd,
		State:     string(r.State),
		ErrorMsg:  r.ErrorMsg,
	}

	res.Head = ServiceLvolToProtoLvol(r.Name, r.Head)
	// spdkrpc.Replica.Snapshots is map[<snapshot name>] rather than map[<snapshot lvol name>]
	for lvolName, lvol := range r.SnapshotLvolMap {
		res.Snapshots[GetSnapshotNameFromReplicaSnapshotLvolName(r.Name, lvolName)] = ServiceLvolToProtoLvol(r.Name, lvol)
	}

	if r.BackingImage != nil {
		backingImageName, _, err := ExtractBackingImageAndDiskUUID(r.BackingImage.Name)
		if err != nil {
			// The BackingImageName will be "" when getting the result from grpc if there is an error.
			// We handle the empty backing image name in the caller.
			// This field is currently only used when engine updating info from replicas or rebuilding the replica.
			r.log.WithError(err).Warnf("Failed to extract backing image name from %v", r.BackingImage.Name)
		}
		res.BackingImageName = backingImageName
	}

	return res
}

func NewReplica(ctx context.Context, replicaName, lvsName, lvsUUID string, specSize uint64, snapshotChecksumEnabled bool, updateCh chan interface{}) *Replica {
	log := logrus.StandardLogger().WithFields(logrus.Fields{
		"replicaName": replicaName,
		"lvsName":     lvsName,
		"lvsUUID":     lvsUUID,
	})

	roundedSpecSize := util.RoundUp(specSize, helpertypes.MiB)
	if roundedSpecSize != specSize {
		log.Infof("Rounded up spec size from %v to %v since the specSize should be multiple of MiB", specSize, roundedSpecSize)
	}
	log = log.WithField("specSize", roundedSpecSize)

	return &Replica{
		ctx: ctx,

		Name:    replicaName,
		Alias:   spdktypes.GetLvolAlias(lvsName, replicaName),
		LvsName: lvsName,
		LvsUUID: lvsUUID,
		Nqn:     helpertypes.GetNQN(replicaName),

		SpecSize: roundedSpecSize,
		State:    types.InstanceStatePending,

		Head: nil,
		ActiveChain: []*Lvol{
			nil,
		},

		SnapshotLvolMap:           map[string]*Lvol{},
		SnapshotLvolHashStatusMap: sync.Map{},
		SnapshotChecksumEnabled:   snapshotChecksumEnabled,

		rebuildingDstCache: RebuildingDstCache{
			rebuildingSnapshotMap: map[string]*api.Lvol{},
			processedSnapshotList: []string{},
		},
		rebuildingSrcCache: RebuildingSrcCache{},

		snapshotCloningSrcCache: map[string]*SnapshotCloningSrcCache{},

		restore: &Restore{},

		UpdateCh: updateCh,

		log: safelog.NewSafeLogger(log),
	}
}

func (r *Replica) GetAddress() string {
	r.RLock()
	defer r.RUnlock()
	return net.JoinHostPort(r.IP, strconv.Itoa(int(r.PortStart)))
}

func (r *Replica) prepareIPAndPorts(portCount int32, superiorPortAllocator *commonbitmap.Bitmap) error {
	podIP, err := commonnet.GetIPForPod()
	if err != nil {
		return err
	}
	r.IP = podIP

	r.PortStart, r.PortEnd, err = superiorPortAllocator.AllocateRange(portCount)
	if err != nil {
		return err
	}

	// Always reserved the 1st port for replica expose and the rest for rebuilding
	bitmap, err := commonbitmap.NewBitmap(r.PortStart+1, r.PortEnd)
	if err != nil {
		return err
	}
	r.portAllocator = bitmap

	r.log.Infof("Prepared IP %s and Ports [%d, %d] for replica", r.IP, r.PortStart, r.PortEnd)

	return nil
}

func (r *Replica) IsRebuilding() bool {
	r.RLock()
	defer r.RUnlock()
	return r.State == types.InstanceStateRunning && r.isRebuilding
}

func (r *Replica) replicaLvolFilter(bdev *spdktypes.BdevInfo) bool {
	if bdev == nil || len(bdev.Aliases) < 1 || bdev.DriverSpecific.Lvol == nil {
		return false
	}
	lvolName := spdktypes.GetLvolNameFromAlias(bdev.Aliases[0])
	// it is okay to have backing image snapshot in the results, because we exclude it when finding root or construct the snapshot map
	return IsReplicaLvol(r.Name, lvolName) || types.IsBackingImageSnapLvolName(lvolName)
}

func (r *Replica) stopSnapshotHash(spdkClient *spdkclient.Client, parentLvol *Lvol) error {
	if parentLvol == nil {
		return nil
	}
	hashStatusValue, exists := r.SnapshotLvolHashStatusMap.Load(parentLvol.Name)
	if !exists {
		return nil
	}
	hashStatus, ok := hashStatusValue.(LvolHashStatus)
	if !ok {
		return nil
	}
	if hashStatus.State == types.ProgressStateInProgress {
		if _, err := spdkClient.BdevLvolStopSnapshotChecksum(parentLvol.Alias); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchProcess(err) {
			return err
		}
		r.SnapshotLvolHashStatusMap.Delete(parentLvol.Name)
		waitSnapshotHashStopped(spdkClient, parentLvol.Alias, r.log)
	}

	return nil
}

// waitSnapshotHashStopped polls BdevLvolStopSnapshotChecksum until SPDK confirms
// that the background checksum goroutine for the given alias has fully stopped
// (indicated by a NoSuchProcess error), guaranteeing that all pending I/O on the
// bdev channel has been drained. It returns when the process is gone, the alias
// is no longer found, or the timeout is exceeded.
func waitSnapshotHashStopped(spdkClient *spdkclient.Client, alias string, log *safelog.SafeLogger) {
	const (
		pollInterval = 500 * time.Millisecond
		timeout      = 10 * time.Second
	)

	deadline := time.Now().Add(timeout)
	for time.Now().Before(deadline) {
		_, err := spdkClient.BdevLvolStopSnapshotChecksum(alias)
		if err == nil {
			// Stop signal delivered; the background task is still running.
			time.Sleep(pollInterval)
			continue
		}
		if jsonrpc.IsJSONRPCRespErrorNoSuchProcess(err) || jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			// SPDK confirmed: no active checksum goroutine remains, I/O is drained.
			return
		}
		log.WithError(err).Warnf("Unexpected error while waiting for snapshot checksum to stop for %s", alias)
		time.Sleep(pollInterval)
	}
	log.Warnf("Timed out waiting for snapshot checksum to stop for %s", alias)
}

func (r *Replica) Sync(spdkClient *spdkclient.Client) (err error) {
	r.Lock()
	defer r.Unlock()
	// It's better to let the server send the update signal

	// This lvol and nvmf subsystem fetch should be protected by replica lock, in case of snapshot operations happened during the sync-up.
	bdevLvolMap, err := GetBdevLvolMapWithFilter(spdkClient, r.replicaLvolFilter)
	if err != nil {
		return err
	}

	if r.State == types.InstanceStatePending {
		return r.construct(bdevLvolMap)
	}

	subsystemMap, err := GetNvmfSubsystemMap(spdkClient)
	if err != nil {
		return err
	}

	return r.validateAndUpdate(bdevLvolMap, subsystemMap)
}

// construct build Replica with the SnapshotLvolMap and SnapshotChain from the bdev lvol list.
// This function is typically invoked for the existing lvols after node/service restart and device add.
func (r *Replica) construct(bdevLvolMap map[string]*spdktypes.BdevInfo) (err error) {
	defer func() {
		if err != nil {
			r.State = types.InstanceStateError
			r.ErrorMsg = err.Error()
		} else {
			if r.State != types.InstanceStateError {
				r.ErrorMsg = ""
			}
		}
	}()

	switch r.State {
	case types.InstanceStatePending:
		break
	case types.InstanceStateStopped:
		if r.reconstructRequired {
			break
		}
		return fmt.Errorf("invalid state %s for reconstructing required flag %v for replica %s construct", r.State, r.reconstructRequired, r.Name)
	case types.InstanceStateRunning:
		if r.isRebuilding {
			break
		}
		fallthrough
	default:
		return fmt.Errorf("invalid state %s with rebuilding %v for replica %s construct", r.State, r.isRebuilding, r.Name)
	}

	if err := r.validateReplicaHead(bdevLvolMap[r.Name]); err != nil {
		return err
	}

	newSnapshotLvolMap, err := constructSnapshotLvolMap(r.Name, bdevLvolMap)
	if err != nil {
		return err
	}
	newChain, err := constructActiveChainFromSnapshotLvolMap(r.Name, newSnapshotLvolMap, bdevLvolMap)
	if err != nil {
		return err
	}

	r.Head = newChain[len(newChain)-1]
	r.ActiveChain = newChain
	r.SnapshotLvolMap = newSnapshotLvolMap
	r.BackingImage = newChain[0]
	r.reconstructRequired = false

	if r.State == types.InstanceStatePending {
		r.State = types.InstanceStateStopped
	}

	return nil
}

func (r *Replica) validateAndUpdate(bdevLvolMap map[string]*spdktypes.BdevInfo, subsystemMap map[string]*spdktypes.NvmfSubsystem) (err error) {
	defer func() {
		if err != nil {
			if r.State != types.InstanceStateError {
				r.State = types.InstanceStateError
				r.log.WithError(err).Error("Found error during validation and update")
			}
			r.ErrorMsg = err.Error()
		} else {
			if r.State != types.InstanceStateError {
				r.ErrorMsg = ""
			}
		}
	}()

	// Stop syncing with the SPDK TGT server if the replica does not contain any valid SPDK components.
	if r.State != types.InstanceStateRunning {
		return nil
	}

	// Should not sync a rebuilding destination replica since the snapshot map as well as the active chain is not ready.
	if r.isRebuilding {
		return nil
	}

	if err := r.validateReplicaHead(bdevLvolMap[r.Name]); err != nil {
		return err
	}

	newSnapshotLvolMap, err := constructSnapshotLvolMap(r.Name, bdevLvolMap)
	if err != nil {
		return err
	}

	// If SnapshotLvolMap is empty but we have snapshots in SPDK, the replica needs reconstruction
	// This can happen after a reboot if construct() wasn't called or failed
	if len(r.SnapshotLvolMap) == 0 && len(newSnapshotLvolMap) > 0 {
		r.log.Warnf("Replica snapshot lvol map is empty but SPDK has %d snapshots, marking for reconstruction", len(newSnapshotLvolMap))
		r.State = types.InstanceStatePending
		return nil
	}

	if len(r.SnapshotLvolMap) != len(newSnapshotLvolMap) {
		return fmt.Errorf("replica current active snapshot lvol map length %d is not the same as the latest snapshot lvol map length %d", len(r.SnapshotLvolMap), len(newSnapshotLvolMap))
	}
	for snapshotLvolName := range r.SnapshotLvolMap {
		if err := r.compareSvcLvols(r.SnapshotLvolMap[snapshotLvolName], newSnapshotLvolMap[snapshotLvolName], true, true); err != nil {
			return err
		}
	}

	newChain, err := constructActiveChainFromSnapshotLvolMap(r.Name, newSnapshotLvolMap, bdevLvolMap)
	if err != nil {
		return err
	}

	if len(r.ActiveChain) != len(newChain) {
		return fmt.Errorf("replica current active chain length %d is not the same as the latest chain length %d", len(r.ActiveChain), len(newChain))
	}

	for idx, svcLvol := range r.ActiveChain {
		newSvcLvol := newChain[idx]
		// Handle nil backing image separately
		if idx == 0 {
			if svcLvol == nil && newSvcLvol == nil {
				continue
			}
			if svcLvol != nil && newSvcLvol == nil {
				return fmt.Errorf("replica current backing image is %v while the latest chain contains a nil backing image", svcLvol.Name)
			}
			if svcLvol == nil && newSvcLvol != nil {
				return fmt.Errorf("replica current backing image is nil while the latest chain contains backing image %v", newSvcLvol.Name)
			}
			// no need to compare the backing image
			continue
		}

		if err := r.compareSvcLvols(svcLvol, newSvcLvol, true, svcLvol.Name != r.Name); err != nil {
			return err
		}
		// Then update the actual size for the head lvol
		if svcLvol.Name == r.Name {
			svcLvol.ActualSize = newSvcLvol.ActualSize
		}
	}

	replicaActualSize := newChain[len(newChain)-1].ActualSize
	for _, snapLvol := range newSnapshotLvolMap {
		replicaActualSize += snapLvol.ActualSize
	}
	r.ActualSize = replicaActualSize

	if r.State == types.InstanceStateRunning {
		if r.IP == "" {
			return fmt.Errorf("found invalid IP %s for replica %s", r.IP, r.Name)
		}
		if r.PortStart == 0 || r.PortEnd == 0 || r.PortStart > r.PortEnd {
			return fmt.Errorf("found invalid Ports [%d, %d] for the running replica %s", r.PortStart, r.PortEnd, r.Name)
		}
	}

	// In case of a stopped replica being wrongly exposed, this function will check the exposing state anyway.
	if r.isRestoring {
		r.log.Info("Replica is being restored, skip the exposing state check")
		return nil
	}

	nqn := helpertypes.GetNQN(r.Name)
	exposedPort, exposedPortErr := getExposedPort(subsystemMap[nqn])
	if r.IsExposed {
		if exposedPortErr != nil {
			return errors.Wrapf(err, "failed to find the actual port in subsystem NQN %s for replica %s, which should be exposed at %d", nqn, r.Name, r.PortStart)
		}
		if exposedPort != r.PortStart {
			return fmt.Errorf("found mismatching between the actual exposed port %d and the recorded port %d for exposed replica %s", exposedPort, r.PortStart, r.Name)
		}
	} else {
		if exposedPortErr == nil {
			return fmt.Errorf("found the actual port %d in subsystem NQN %s for replica %s, which should not be exposed", exposedPort, nqn, r.Name)
		}
	}

	return nil
}

func (r *Replica) compareSvcLvols(prev, cur *Lvol, checkChildren, checkActualSize bool) error {
	if prev == nil && cur == nil {
		return nil
	}
	if prev == nil {
		return fmt.Errorf("cannot find the corresponding prev lvol")
	}
	if cur == nil {
		return fmt.Errorf("cannot find the corresponding cur lvol")
	}
	if prev.Name != cur.Name ||
		prev.UUID != cur.UUID ||
		prev.SnapshotTimestamp != cur.SnapshotTimestamp ||
		prev.SpecSize != cur.SpecSize ||
		// TODO: handle parent changing case
		//prev.Parent != cur.Parent ||
		len(prev.Children) != len(cur.Children) {
		return fmt.Errorf("found mismatching lvol %+v with recorded prev lvol %+v", cur, prev)
	}
	if checkChildren {
		for childName := range prev.Children {
			if cur.Children[childName] == nil {
				return fmt.Errorf("found mismatching lvol children %+v with recorded prev lvol children %+v when validating lvol %s", cur.Children, prev.Children, prev.Name)
			}
		}
	}

	// TODO:
	// When deleting a snapshot lvol, the merge of lvols results in a change of actual size. Do not return error to prevent a false alarm.
	// Need to revisit the actual size check.
	if checkActualSize && prev.ActualSize != cur.ActualSize {
		r.log.Warnf("Found mismatching lvol actual size %v with recorded prev lvol actual size %v when validating lvol %s", cur.ActualSize, prev.ActualSize, prev.Name)
	}

	r.SyncSnapshotHashStatus(cur)
	prev.SnapshotChecksum = cur.SnapshotChecksum

	return nil
}

func (r *Replica) SyncSnapshotHashStatus(snapSvcLvol *Lvol) {
	if snapSvcLvol == nil {
		return
	}

	var hashStatus LvolHashStatus
	hashStatusValue, hashStatusExists := r.SnapshotLvolHashStatusMap.Load(snapSvcLvol.Name)
	if hashStatusExists {
		hashStatus = hashStatusValue.(LvolHashStatus)
	}
	if snapSvcLvol.SnapshotChecksum != "" {
		hashStatus.State = types.ProgressStateComplete
		hashStatus.Checksum = snapSvcLvol.SnapshotChecksum
		hashStatus.Error = ""
		r.SnapshotLvolHashStatusMap.Store(snapSvcLvol.Name, hashStatus)
	} else {
		// If the snapshot checksum hashing may be in-progress or failed, there is no need to clean up the status cache.
		if hashStatus.State == types.ProgressStateComplete || hashStatus.State == types.ProgressStateError {
			r.SnapshotLvolHashStatusMap.Delete(snapSvcLvol.Name)
		}
	}
}

func getExposedPort(subsystem *spdktypes.NvmfSubsystem) (exposedPort int32, err error) {
	if subsystem == nil || len(subsystem.ListenAddresses) == 0 {
		return 0, fmt.Errorf("cannot find the NVMf subsystem")
	}

	port := 0
	for _, listenAddr := range subsystem.ListenAddresses {
		if !strings.EqualFold(string(listenAddr.Adrfam), string(spdktypes.NvmeAddressFamilyIPv4)) ||
			!strings.EqualFold(string(listenAddr.Trtype), string(spdktypes.NvmeTransportTypeTCP)) {
			continue
		}
		port, err = strconv.Atoi(listenAddr.Trsvcid)
		if err != nil {
			return 0, err
		}
		return int32(port), nil
	}

	return 0, fmt.Errorf("cannot find a exposed port in the NVMf subsystem")
}

func (r *Replica) validateReplicaHead(headBdevLvol *spdktypes.BdevInfo) (err error) {
	if headBdevLvol == nil {
		return fmt.Errorf("found nil head bdev lvol for replica %s", r.Name)
	}
	if headBdevLvol.DriverSpecific.Lvol.Snapshot {
		return fmt.Errorf("found the head bdev lvol is a snapshot lvol for replica %s", r.Name)
	}
	if r.LvsUUID != headBdevLvol.DriverSpecific.Lvol.LvolStoreUUID {
		return fmt.Errorf("found mismatching lvol LvsUUID %v with recorded LvsUUID %v for replica %s", headBdevLvol.DriverSpecific.Lvol.LvolStoreUUID, r.LvsUUID, r.Name)
	}
	bdevLvolSpecSize := headBdevLvol.NumBlocks * uint64(headBdevLvol.BlockSize)
	if r.SpecSize != 0 && r.SpecSize < bdevLvolSpecSize {
		return fmt.Errorf("found mismatching lvol spec size %v with recorded spec size %v for replica %s", bdevLvolSpecSize, r.SpecSize, r.Name)
	}

	return nil
}

func (r *Replica) ensureValidHeadLvol(spdkClient *spdkclient.Client) (bool, error) {
	headBdevLvol, err := spdkClient.BdevLvolGetByName(r.Alias, 0)
	if err != nil {
		if !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return false, err
		}
		return false, nil
	}

	if validateErr := r.validateReplicaHead(&headBdevLvol); validateErr != nil {
		r.log.WithError(validateErr).Warnf("Found invalid head lvol %v for replica %v, will delete it first", headBdevLvol.Name, r.Name)
		if _, deleteErr := spdkClient.BdevLvolDelete(headBdevLvol.UUID); deleteErr != nil {
			return false, errors.Wrapf(deleteErr, "failed to delete invalid head lvol %v for replica %v whose validation error is: %v",
				headBdevLvol.Name, r.Name, validateErr)
		}
		return false, nil
	}

	return true, nil
}

func (r *Replica) updateHeadCache(spdkClient *spdkclient.Client) error {
	headBdevLvol, err := spdkClient.BdevLvolGetByName(r.Alias, 0)
	if err != nil {
		return err
	}

	r.Head = BdevLvolInfoToServiceLvol(&headBdevLvol)

	r.attachHeadToActiveChain()
	return r.linkHeadWithParent()
}

func (r *Replica) attachHeadToActiveChain() {
	if r.shouldAppendHeadToActiveChain() {
		r.ActiveChain = append(r.ActiveChain, r.Head)
		return
	}

	// Replace the existing head entry
	r.ActiveChain[len(r.ActiveChain)-1] = r.Head
}

func (r *Replica) shouldAppendHeadToActiveChain() bool {
	if len(r.ActiveChain) == 1 {
		return true
	}

	last := r.ActiveChain[len(r.ActiveChain)-1]
	return last != nil && last.Name != r.Name
}

func (r *Replica) linkHeadWithParent() error {
	parentIndex := len(r.ActiveChain) - 2
	if parentIndex < 0 {
		return fmt.Errorf("invalid active chain length %d when updating head cache", len(r.ActiveChain))
	}

	if parentIndex == 0 && r.BackingImage != nil {
		r.BackingImage.Lock()
		defer r.BackingImage.Unlock()
	}

	parent := r.ActiveChain[parentIndex]
	if parent == nil {
		return nil
	}

	if parent.Name != r.Head.Parent {
		return fmt.Errorf("found active chain parent %v does not match head parent %v", parent.Name, r.Head.Parent)
	}

	parent.Children[r.Head.Name] = r.Head
	return nil
}

// func (r *Replica) updateHeadCache(spdkClient *spdkclient.Client) (err error) {
// 	headBdevLvol, err := spdkClient.BdevLvolGetByName(r.Alias, 0)
// 	if err != nil {
// 		return err
// 	}
// 	r.Head = BdevLvolInfoToServiceLvol(&headBdevLvol)

// 	if len(r.ActiveChain) == 1 || (r.ActiveChain[len(r.ActiveChain)-1] != nil && r.ActiveChain[len(r.ActiveChain)-1].Name != r.Name) {
// 		r.ActiveChain = append(r.ActiveChain, r.Head)
// 	} else {
// 		r.ActiveChain[len(r.ActiveChain)-1] = r.Head
// 	}

// 	index := len(r.ActiveChain) - 2
// 	if index < 0 {
// 		return fmt.Errorf("invalid active chain length %d when updating head cache", len(r.ActiveChain))
// 	}
// 	if index == 0 && r.BackingImage != nil {
// 		r.BackingImage.Lock()
// 		defer r.BackingImage.Unlock()
// 	}
// 	if r.ActiveChain[index] != nil {
// 		if r.ActiveChain[index].Name != r.Head.Parent {
// 			return fmt.Errorf("found the last entry of the active chain %v is not the head parent %v", r.ActiveChain[index].Name, r.Head.Parent)
// 		}
// 		r.ActiveChain[index].Children[r.Head.Name] = r.Head
// 	}

// 	return nil
// }

// prepareHead ensures the replica head lvol exists, is valid, and is correctly
// attached to the active lvol chain.
//
// This function is responsible for the full lifecycle of the replica head lvol,
// including validation, cleanup, reuse, recreation, and cache synchronization.
//
//  1. Validate the existing head lvol.
//     - If the head lvol does not exist, it will be created later.
//     - If the head lvol exists but is invalid, it will be deleted.
//     - If the head lvol exists and is valid, it will be reused.
//  2. Apply the backing image (if provided) as the root of the ActiveChain.
//  3. If the head lvol is unavailable:
//     - Determine the correct parent lvol from ActiveChain.
//     - Clone a new head lvol from the parent if one exists, otherwise create a brand-new head lvol.
//     - Resize the head lvol if the parent size differs from the replica spec size.
//  4. Clean up any stale head entry from ActiveChain.
//  5. Update the in-memory head cache to reflect the actual SPDK state.
//
// After this function returns successfully:
//   - A valid head lvol always exists in SPDK.
//   - r.Head is populated and consistent with SPDK.
//   - r.ActiveChain does not include the head lvol itself.
func (r *Replica) prepareHead(spdkClient *spdkclient.Client, backingImage *BackingImage) error {
	headIsAvailable, err := r.ensureValidHeadLvol(spdkClient)
	if err != nil {
		return err
	}

	if backingImage != nil {
		r.ActiveChain[0] = backingImage.Snapshot
		r.BackingImage = r.ActiveChain[0]
		r.log.WithField("backingImage", backingImage.Name)
	}

	if !headIsAvailable {
		var headParentLvol *Lvol
		if r.ActiveChain[len(r.ActiveChain)-1] != nil {
			if r.ActiveChain[len(r.ActiveChain)-1].Name == r.Name {
				if len(r.ActiveChain) < 2 {
					return fmt.Errorf("found invalid active chain %+v when preparing head for replica %s", len(r.ActiveChain), r.Name)
				}
				headParentLvol = r.ActiveChain[len(r.ActiveChain)-2]
			} else {
				headParentLvol = r.ActiveChain[len(r.ActiveChain)-1]
			}
		} else {
			if len(r.ActiveChain) > 1 { // The only possible case is that r.ActiveChain[len(r.ActiveChain)-1] is a nil head
				r.ActiveChain = r.ActiveChain[:len(r.ActiveChain)-1]
				headParentLvol = r.ActiveChain[len(r.ActiveChain)-1]
			}
		}
		if headParentLvol != nil { // The replica has a backing image or somehow there are already snapshots in the chain
			if _, err := spdkClient.BdevLvolClone(headParentLvol.Alias, r.Name); err != nil {
				return err
			}
			if headParentLvol.SpecSize != r.SpecSize {
				if _, err := spdkClient.BdevLvolResize(r.Alias, util.BytesToMiB(r.SpecSize)); err != nil {
					return err
				}
			}
			r.log.Infof("Replica cloned a new head lvol from the parent lvol %s", headParentLvol.Name)
		} else {
			if _, err := spdkClient.BdevLvolCreate("", r.LvsUUID, r.Name, util.BytesToMiB(r.SpecSize), "", true); err != nil {
				return err
			}
			r.log.Info("Replica created a new head lvol")
		}
	} else {
		headBdevLvol, err := spdkClient.BdevLvolGetByName(r.Alias, 0)
		if err != nil {
			return err
		}
		headSpecSize := headBdevLvol.NumBlocks * uint64(headBdevLvol.BlockSize)
		if headSpecSize < r.SpecSize {
			if _, err := spdkClient.BdevLvolResize(r.Alias, util.BytesToMiB(r.SpecSize)); err != nil {
				return err
			}
			r.log.Infof("Replica resized the existing head lvol from %d to %d before reuse", headSpecSize, r.SpecSize)
		}

		// The head lvol is already available, so we need to update the head cache
		r.log.Info("Replica head lvol is already available, will directly reuse it")
	}

	// Blindly clean up then update the caches for the head
	r.Head = nil
	r.removeHeadFromActiveChainIfExists()

	return r.updateHeadCache(spdkClient)
}

func (r *Replica) removeHeadFromActiveChainIfExists() {
	if len(r.ActiveChain) == 0 {
		return
	}

	last := r.ActiveChain[len(r.ActiveChain)-1]
	if last == nil || last.Name != r.Name {
		return
	}

	r.ActiveChain = r.ActiveChain[:len(r.ActiveChain)-1]
}

func (r *Replica) validateAndSyncLvstore(spdkClient *spdkclient.Client) error {
	var (
		lvsList []spdktypes.LvstoreInfo
		err     error
	)

	if r.LvsUUID != "" {
		lvsList, err = spdkClient.BdevLvolGetLvstore("", r.LvsUUID)
	} else if r.LvsName != "" {
		lvsList, err = spdkClient.BdevLvolGetLvstore(r.LvsName, "")
	}
	if err != nil {
		return err
	}
	if len(lvsList) != 1 {
		return fmt.Errorf("found zero or multiple lvstore with name %s and UUID %s during replica %s creation", r.LvsName, r.LvsUUID, r.Name)
	}
	if r.LvsName == "" {
		r.LvsName = lvsList[0].Name
	}
	if r.LvsUUID == "" {
		r.LvsUUID = lvsList[0].UUID
	}
	if r.LvsName != lvsList[0].Name || r.LvsUUID != lvsList[0].UUID {
		return fmt.Errorf("found mismatching between the actual lvstore name %s with UUID %s and the recorded lvstore name %s with UUID %s during replica %s creation", lvsList[0].Name, lvsList[0].UUID, r.LvsName, r.LvsUUID, r.Name)
	}

	return nil
}

// getRootLvolName relies on the lvol name to identify if a lvol belongs to the replica,
// then figuring out whether it is the root by checking the parent
func getRootLvolName(replicaName string, bdevLvolMap map[string]*spdktypes.BdevInfo) (rootLvolName string) {
	for lvolName, bdevLvol := range bdevLvolMap {
		if lvolName != replicaName && !IsReplicaSnapshotLvol(replicaName, lvolName) {
			continue
		}
		// Consider that a backing image can be the parent of the replica root
		if bdevLvol.DriverSpecific.Lvol.BaseSnapshot != "" && IsReplicaSnapshotLvol(replicaName, bdevLvol.DriverSpecific.Lvol.BaseSnapshot) {
			continue
		}
		return lvolName
	}

	return ""
}

func constructSnapshotLvolMap(replicaName string, bdevLvolMap map[string]*spdktypes.BdevInfo) (res map[string]*Lvol, err error) {
	rootLvolName := getRootLvolName(replicaName, bdevLvolMap)
	if rootLvolName == "" {
		return nil, fmt.Errorf("cannot find the root of the replica during snapshot lvol map construction")
	}
	res = map[string]*Lvol{}

	queue := []*Lvol{BdevLvolInfoToServiceLvol(bdevLvolMap[rootLvolName])}
	for ; len(queue) > 0; queue = queue[1:] {
		curSvcLvol := queue[0]
		if curSvcLvol == nil || curSvcLvol.Name == replicaName {
			continue
		}
		if !IsReplicaSnapshotLvol(replicaName, curSvcLvol.Name) {
			continue
		}
		res[curSvcLvol.Name] = curSvcLvol

		if bdevLvolMap[curSvcLvol.Name].DriverSpecific.Lvol.Clones == nil {
			continue
		}
		for _, childLvolName := range bdevLvolMap[curSvcLvol.Name].DriverSpecific.Lvol.Clones {
			// Exclude the children lvols that does not belong to this replica. For example, the leftover rebuilding lvols of the previous rebuilding failed replicas
			// or linked-clone lvol of another replica
			if !IsReplicaLvol(replicaName, childLvolName) {
				delete(curSvcLvol.Children, childLvolName)
				continue
			}
			if bdevLvolMap[childLvolName] == nil {
				return nil, fmt.Errorf("cannot find child lvol %v for lvol %v during the snapshot lvol map construction", childLvolName, curSvcLvol.Name)
			}
			curSvcLvol.Children[childLvolName] = BdevLvolInfoToServiceLvol(bdevLvolMap[childLvolName])
			queue = append(queue, curSvcLvol.Children[childLvolName])
		}
	}

	return res, nil
}

// constructActiveChainFromSnapshotLvolMap retrieves the chain bottom up (from the head to the ancestor snapshot/backing image).
func constructActiveChainFromSnapshotLvolMap(replicaName string, snapshotLvolMap map[string]*Lvol, bdevLvolMap map[string]*spdktypes.BdevInfo) (res []*Lvol, err error) {
	headBdevLvol := bdevLvolMap[replicaName]
	if headBdevLvol == nil {
		return nil, fmt.Errorf("found nil head bdev lvol for replica %s", replicaName)
	}

	var headSvcLvol *Lvol
	headParentSnapshotLvolName := headBdevLvol.DriverSpecific.Lvol.BaseSnapshot
	if IsReplicaSnapshotLvol(replicaName, headParentSnapshotLvolName) {
		headParentSnapSvcLvol := snapshotLvolMap[headParentSnapshotLvolName]
		if headParentSnapSvcLvol == nil {
			return nil, fmt.Errorf("cannot find the parent snapshot %s of the head for replica %s", headParentSnapshotLvolName, replicaName)
		}
		headSvcLvol = headParentSnapSvcLvol.Children[replicaName]
	} else { // The parent of the head is nil or a backing image
		headSvcLvol = BdevLvolInfoToServiceLvol(headBdevLvol)
	}
	if headSvcLvol == nil {
		return nil, fmt.Errorf("found nil head svc lvol for replica %s", replicaName)
	}

	newChain := []*Lvol{headSvcLvol}
	// TODO: Considering the clone, this function or `constructSnapshotMap` may need to construct the children map for the head

	// Build the majority of the chain with `snapshotMap` so that it does not need to worry about the snap svc lvol children map maintenance.
	for curSvcLvol := snapshotLvolMap[headSvcLvol.Parent]; curSvcLvol != nil; curSvcLvol = snapshotLvolMap[curSvcLvol.Parent] {
		newChain = append(newChain, curSvcLvol)
	}

	// Check if the root snap/head lvol has a parent. If YES, it means that this replica contains a backing image or
	// this replica is linked-cloned from another replica
	var biSvcLvol *Lvol
	rootLvol := newChain[len(newChain)-1]
	if rootLvol.Parent != "" && types.IsBackingImageSnapLvolName(rootLvol.Parent) {
		// Here we won't maintain the complete children map for the backing image Lvol since it may contain root lvols of other replicas
		biBdevLvol := bdevLvolMap[rootLvol.Parent]
		if biBdevLvol == nil {
			return nil, fmt.Errorf("cannot find backing image lvol %v for the current bdev lvol map for replica %s", rootLvol.Parent, replicaName)
		}
		biSvcLvol = BdevLvolInfoToServiceLvol(biBdevLvol)
		biSvcLvol.Children[rootLvol.Name] = rootLvol
	}
	newChain = append(newChain, biSvcLvol)

	// Need to flip r.ActiveSnapshotChain. By convention the oldest one (backing image) should be at index 0
	for head, tail := 0, len(newChain)-1; head < tail; head, tail = head+1, tail-1 {
		newChain[head], newChain[tail] = newChain[tail], newChain[head]
	}

	return newChain, nil
}

// Create initiates the replica, prepares the head lvol bdev then blindly exposes it for the replica.
func (r *Replica) Create(spdkClient *spdkclient.Client, portCount int32, superiorPortAllocator *commonbitmap.Bitmap, backingImage *BackingImage) (ret *spdkrpc.Replica, err error) {
	updateRequired := true

	r.Lock()
	defer func() {
		r.Unlock()

		if updateRequired {
			r.UpdateCh <- nil
		}
	}()

	if r.State == types.InstanceStateRunning {
		updateRequired = false
		return nil, grpcstatus.Errorf(grpccodes.AlreadyExists, "replica %v already exists and running", r.Name)
	}
	if r.State != types.InstanceStatePending && r.State != types.InstanceStateStopped {
		updateRequired = false
		return nil, grpcstatus.Errorf(grpccodes.FailedPrecondition, "invalid state %s for replica %s creation", r.State, r.Name)
	}

	defer func() {
		if err != nil {
			r.log.WithError(err).Errorf("Failed to create replica %s", r.Name)

			// Set the replica state to error. longhorn-manager controller will be aware of this and take actions.
			r.State = types.InstanceStateError
			r.ErrorMsg = err.Error()

			ret = ServiceReplicaToProtoReplica(r)
			err = nil
		} else {
			// Don't override error state if the replica is already in error state
			// The error state may be set in construct() or validateAndUpdate()
			if r.State == types.InstanceStateError {
				return
			}

			r.ErrorMsg = ""
			r.log.Info("Created replica")
		}
	}()

	// Create bdev lvol if the replica is the new one
	if r.State == types.InstanceStatePending {
		if len(r.ActiveChain) != 1 {
			return nil, fmt.Errorf("invalid chain length %d for new replica creation", len(r.ActiveChain))
		}
	}

	if err := r.validateAndSyncLvstore(spdkClient); err != nil {
		return nil, err
	}

	// A stopped replica may be a broken one. We need to make sure the head lvol is ready first.
	if err := r.prepareHead(spdkClient, backingImage); err != nil {
		return nil, err
	}

	// In case of failed replica reuse/restart being errored by r.validateAndUpdate(), we should make sure the caches are correct.
	// Also handle the case where an existing replica was discovered after reboot but construct() wasn't called yet.
	if r.State == types.InstanceStatePending && (r.reconstructRequired || len(r.SnapshotLvolMap) == 0) {
		bdevLvolMap, err := GetBdevLvolMapWithFilter(spdkClient, r.replicaLvolFilter)
		if err != nil {
			return nil, err
		}
		r.log.Info("Constructing replica state from SPDK for pending replica")
		if err := r.construct(bdevLvolMap); err != nil {
			return nil, err
		}
		r.State = types.InstanceStateStopped
	}

	if err := r.prepareIPAndPorts(portCount, superiorPortAllocator); err != nil {
		return nil, err
	}

	// Blindly stop exposing the bdev if it exists. This is to avoid potential inconsistencies during salvage case.
	r.log.Infof("Stopping exposing bdev for replica creation if it is already exposed to avoid potential inconsistency during salvage case")
	if err := spdkClient.StopExposeBdev(r.Nqn); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return nil, errors.Wrapf(err, "failed to stop exposing replica %v", r.Name)
	}

	if err := spdkClient.StartExposeBdev(r.Nqn, r.Head.UUID, generateNGUID(r.Name), r.IP, strconv.Itoa(int(r.PortStart))); err != nil {
		return nil, err
	}

	r.IsExposed = true
	r.State = types.InstanceStateRunning

	return ServiceReplicaToProtoReplica(r), nil
}

func (r *Replica) stopAllSnapshotHashing(spdkClient *spdkclient.Client) error {
	var errs error

	for snapLvolName, snapLvol := range r.SnapshotLvolMap {
		if err := r.stopSnapshotHash(spdkClient, snapLvol); err != nil {
			errs = multierr.Append(errs, errors.Wrapf(err, "failed to stop snapshot %s checksum hashing", snapLvolName))
		}
	}

	return errs
}

func (r *Replica) cleanupLvolTrees(spdkClient *spdkclient.Client) error {
	if len(r.ActiveChain) <= 1 {
		return nil
	}

	bdevLvolMap, err := GetBdevLvolMapWithFilter(spdkClient, r.replicaLvolFilter)
	if err != nil {
		return err
	}

	for lvolName, bdevLvol := range bdevLvolMap {
		if types.IsBackingImageSnapLvolName(lvolName) {
			for _, childLvolName := range bdevLvol.DriverSpecific.Lvol.Clones {
				if !IsReplicaLvol(r.Name, childLvolName) {
					continue
				}
				r.CleanupLvolTree(spdkClient, childLvolName, bdevLvolMap)
			}
			continue
		}
		r.CleanupLvolTree(spdkClient, lvolName, bdevLvolMap)
	}

	return nil
}

func (r *Replica) Delete(spdkClient *spdkclient.Client, cleanupRequired bool, superiorPortAllocator *commonbitmap.Bitmap) (err error) {
	updateRequired := false

	r.Lock()
	defer func() {
		// Considering that there may be still pending validations, it's better to update the state after the deletion.
		prevState := r.State
		if err != nil {
			r.log.WithError(err).Errorf("Failed to delete replica with cleanupRequired flag %v", cleanupRequired)
			if r.isRestoring {
				// This is not a real error. No need to update the state.
			} else if r.State != types.InstanceStateError {
				r.State = types.InstanceStateError
				r.ErrorMsg = err.Error()
			}
		} else {
			if r.State == types.InstanceStatePending {
				if cleanupRequired {
					r.State = types.InstanceStateTerminating
				}
			} else if r.State != types.InstanceStateTerminating {
				if !r.isRestoring {
					if cleanupRequired {
						r.State = types.InstanceStateTerminating
					} else {
						r.State = types.InstanceStateStopped
					}
				}
			}
		}

		if r.State != types.InstanceStateError {
			r.ErrorMsg = ""
		}

		if prevState == types.InstanceStateError {
			r.reconstructRequired = true
		}

		if prevState != r.State {
			updateRequired = true
			r.log.Infof("Replica state changed from %s to %s after deletion with cleanupRequired flag %v", prevState, r.State, cleanupRequired)
		}

		r.Unlock()

		if updateRequired {
			r.UpdateCh <- nil
		}
	}()

	if r.State == types.InstanceStatePending && !cleanupRequired {
		// A pending replica without cleanup is a no-op
		r.log.Info("Skipped deletion for a pending replica as cleanup is not required")
		return nil
	}

	if r.isRestoring && r.restore != nil {
		r.log.Info("Canceling volume restoration before replica deletion")
		r.restore.Stop()
		return fmt.Errorf("waiting for volume restoration to stop")
	}

	if err := r.stopAllSnapshotHashing(spdkClient); err != nil {
		return errors.Wrapf(err, "failed to stop all snapshot hashing before replica deletion with cleanupRequired %v", cleanupRequired)
	}

	if r.IsExposed {
		r.log.Info("Unexposing bdev for replica deletion")
		if err := spdkClient.StopExposeBdev(helpertypes.GetNQN(r.Name)); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return err
		}
		r.IsExposed = false
		updateRequired = true
	}

	// Clean up the rebuilding cached info first
	r.log.Info("Cleaning up the rebuilding src")
	r.doCleanupForRebuildingSrc(spdkClient)

	r.log.Info("Cleaning up the rebuilding dst")
	_ = r.doCleanupForRebuildingDst(spdkClient)
	if r.isRebuilding {
		r.rebuildingDstCache.rebuildingError = "replica is being deleted"
		r.rebuildingDstCache.rebuildingState = types.ProgressStateError
		r.isRebuilding = false
	}

	// Clean up the cloning cached info
	if r.snapshotCloningDstCache.monitorCancelFunc != nil {
		r.snapshotCloningDstCache.monitorCancelFunc()
		r.snapshotCloningDstCache.monitorCancelFunc = nil
	}

	r.log.Info("Cleaning up the snapshot cloning dst")
	if err := r.doCleanupForSnapshotCloneDst(spdkClient, false); err != nil {
		r.log.WithError(err).Error("Failed to delete replica")
	}
	if r.isSnapshotCloning {
		if r.snapshotCloningDstCache.cloningState != types.ProgressStateError {
			r.snapshotCloningDstCache.cloningError = "replica is being deleted"
			r.snapshotCloningDstCache.cloningState = types.ProgressStateError
		}
		r.isSnapshotCloning = false
	}

	// The port can be released once the rebuilding and expose are stopped.
	if r.PortStart != 0 {
		if err := superiorPortAllocator.ReleaseRange(r.PortStart, r.PortEnd); err != nil {
			return errors.Wrapf(err, "failed to release port %d to %d during replica deletion with cleanup flag %v", r.PortStart, r.PortEnd, cleanupRequired)
		}
		r.portAllocator = nil
		r.PortStart, r.PortEnd = 0, 0
		updateRequired = true
	}

	if !cleanupRequired {
		return nil
	}

	// Use r.Alias here since we don't know if an errored replicas still contains the head lvol
	if _, err := spdkClient.BdevLvolDelete(r.Alias); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return err
	}

	updateRequired = true

	// Clean up the valid snapshot tree as well as all possible leftovers or out of track lvols
	r.log.Info("Cleaning up the snapshot tree")
	if err := r.cleanupLvolTrees(spdkClient); err != nil {
		return errors.Wrapf(err, "failed to clean up snapshots during replica %s deletion", r.Name)
	}

	r.log.Info("Deleted replica with all possible lvols")

	return nil
}

func (r *Replica) Get() (pReplica *spdkrpc.Replica) {
	r.RLock()
	defer r.RUnlock()

	return ServiceReplicaToProtoReplica(r)
}

func (r *Replica) Expand(spdkClient *spdkclient.Client, size uint64) error {
	r.Lock()
	defer r.Unlock()

	r.log.Infof("Expanding replica %s to size %v", r.Name, size)

	clusterSize, err := r.fetchClusterSize(spdkClient)
	if err != nil {
		return errors.Wrapf(err, "failed to fetch cluster size for replica %v", r.Name)
	}

	roundedSize := util.RoundUp(size, clusterSize)
	if roundedSize != size {
		return fmt.Errorf("replica %s rounded up spec size from %v to %v since the spec size should be multiple of MiB", r.Name, size, roundedSize)
	}

	if r.SpecSize > size {
		return fmt.Errorf("cannot expand replica %s to a smaller size %v, current spec size %v", r.Name, size, r.SpecSize)
	}
	if r.SpecSize == size {
		r.log.Infof("Replica %s had been expanded to size %v", r.Name, size)
		return nil
	}

	// If the bdev is exposed, we must stop exposing it before the resize.
	reExposeBdev := false
	if r.IsExposed {
		if err := spdkClient.StopExposeBdev(helpertypes.GetNQN(r.Name)); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return errors.Wrapf(err, "failed to stop expose replica %v before expansion", r.Name)
		}
		r.IsExposed = false
		reExposeBdev = true
	}

	resized, err := spdkClient.BdevLvolResize(r.Alias, util.BytesToMiB(size))
	if !resized || err != nil {
		r.log.Warn("Failed to expand replica or returned false; verifying lvol size")

		// Some replicas may have returned an error during expansion due to unexpected issues
		// (e.g. temporary network glitch, internal error, timeout).
		// To avoid mistakenly marking those as failed, we perform a single follow-up check
		// to verify if the replica was actually expanded.

		headBdevLvol, getLvolErr := spdkClient.BdevLvolGetByName(r.Alias, 0)
		if getLvolErr != nil {
			return errors.Wrapf(err, "failed to get bdev lvol: %v", getLvolErr)
		}
		lvol := BdevLvolInfoToServiceLvol(&headBdevLvol)
		if lvol.SpecSize != size {
			if err != nil {
				return errors.Wrapf(err, "bdev lvol resize error")
			}

			if !resized {
				return fmt.Errorf("no error, but replica %s not resized", r.Name)
			}
		}

		r.log.Info("Replica expansion succeeded despite earlier error")
	}

	// If we had previously exposed the bdev, we must re-expose it after the resize.
	if reExposeBdev {
		if err := spdkClient.StartExposeBdev(helpertypes.GetNQN(r.Name), r.Head.UUID, generateNGUID(r.Name), r.IP, strconv.Itoa(int(r.PortStart))); err != nil {
			return errors.Wrapf(err, "failed to start expose replica %v after expansion", r.Name)
		}
		r.IsExposed = true
	}

	// Blindly clean up then update the caches for the head
	r.Head = nil
	if len(r.ActiveChain) > 0 &&
		r.ActiveChain[len(r.ActiveChain)-1] != nil &&
		r.ActiveChain[len(r.ActiveChain)-1].Name == r.Name {
		r.ActiveChain = r.ActiveChain[:len(r.ActiveChain)-1]
	}

	if err := r.updateHeadCache(spdkClient); err != nil {
		return errors.Wrapf(err, "failed to update head cache for replica %v", r.Name)
	}

	r.log.Info("Expanding replica complete")
	r.SpecSize = size
	return nil
}

func (r *Replica) fetchClusterSize(spdkClient *spdkclient.Client) (uint64, error) {
	var (
		lvsList []spdktypes.LvstoreInfo
		err     error
	)

	switch {
	case r.LvsUUID != "":
		lvsList, err = spdkClient.BdevLvolGetLvstore("", r.LvsUUID)
	case r.LvsName != "":
		lvsList, err = spdkClient.BdevLvolGetLvstore(r.LvsName, "")
	default:
		return 0, fmt.Errorf("either LvsUUID or LvsName must be set for replica %s", r.Name)
	}

	if err != nil {
		return 0, errors.Wrapf(err, "failed to query lvstore for replica %s (name=%s uuid=%s)", r.Name, r.LvsName, r.LvsUUID)
	}

	if len(lvsList) != 1 {
		return 0, fmt.Errorf("unexpected number of lvstores (%d) found for replica %s (name=%s uuid=%s)", len(lvsList), r.Name, r.LvsName, r.LvsUUID)
	}

	return lvsList[0].ClusterSize, nil
}

func getSnapshotXattrsFromOptions(opts *api.SnapshotOptions) []spdkclient.Xattr {
	if opts == nil {
		return nil
	}

	return []spdkclient.Xattr{
		{
			Name:  spdkclient.UserCreated,
			Value: strconv.FormatBool(opts.UserCreated),
		},
		{
			Name:  spdkclient.SnapshotTimestamp,
			Value: opts.Timestamp,
		},
	}
}

func (r *Replica) SnapshotCreate(spdkClient *spdkclient.Client, snapshotName string, opts *api.SnapshotOptions) (replica *spdkrpc.Replica, err error) {
	updateRequired := false

	r.Lock()
	defer func() {
		r.Unlock()

		if updateRequired {
			r.UpdateCh <- nil
		}
	}()

	r.log.Infof("Creating snapshot %s with options %+v for replica %s", snapshotName, opts, r.Name)

	if r.State != types.InstanceStateStopped && r.State != types.InstanceStateRunning {
		return nil, fmt.Errorf("invalid state %v for replica %s snapshot creation", r.State, r.Name)
	}

	snapLvolName := GetReplicaSnapshotLvolName(r.Name, snapshotName)
	if _, exists := r.SnapshotLvolMap[snapLvolName]; exists {
		return nil, fmt.Errorf("snapshot %s(%s) already exists in replica %s", snapshotName, snapLvolName, r.Name)
	}

	defer func() {
		if err != nil {
			if r.State != types.InstanceStateError {
				r.State = types.InstanceStateError
				updateRequired = true
			}
			r.ErrorMsg = err.Error()
			r.log.WithError(err).Errorf("Failed to create snapshot %s for replica %s", snapshotName, r.Name)
		} else {
			if r.State != types.InstanceStateError {
				r.ErrorMsg = ""
			}
		}
	}()

	if r.Head == nil {
		return nil, fmt.Errorf("nil head for replica snapshot creation")
	}

	xattrs := getSnapshotXattrsFromOptions(opts)

	snapUUID, err := spdkClient.BdevLvolSnapshot(r.Head.UUID, snapLvolName, xattrs)
	if err != nil {
		return nil, err
	}

	snapBdevLvol, err := spdkClient.BdevLvolGetByName(snapUUID, 0)
	if err != nil {
		return nil, err
	}
	snapSvcLvol := BdevLvolInfoToServiceLvol(&snapBdevLvol)

	headBdevLvol, err := spdkClient.BdevLvolGetByName(r.Head.UUID, 0)
	if err != nil {
		return nil, err
	}
	r.Head = BdevLvolInfoToServiceLvol(&headBdevLvol)
	snapSvcLvol.Children[r.Head.Name] = r.Head

	// Rewire the in-memory chain/children links to insert the new snapshot between
	// the previous parent lvol (if any) and the refreshed head.
	//
	// Before:
	//   ActiveChain: [..., prev, head]
	//   prev.Children[head] = head
	//
	// After:
	//   ActiveChain: [..., prev, snap, head]
	//   prev.Children[snap] = snap
	//   snap.Children[head] = head
	//
	// If there is no prev lvol (i.e. the chain only contained the head), we only
	// need to build the snap -> head link and update ActiveChain accordingly.
	if len(r.ActiveChain) > 1 && r.ActiveChain[len(r.ActiveChain)-2] != nil {
		prevSvcLvol := r.ActiveChain[len(r.ActiveChain)-2]
		prevSvcLvol.Lock()
		delete(prevSvcLvol.Children, r.Head.Name)
		prevSvcLvol.Children[snapSvcLvol.Name] = snapSvcLvol
		prevSvcLvol.Unlock()
	}
	r.ActiveChain[len(r.ActiveChain)-1] = snapSvcLvol
	r.ActiveChain = append(r.ActiveChain, r.Head)
	r.SnapshotLvolMap[snapLvolName] = snapSvcLvol
	updateRequired = true

	r.log.Infof("Replica created snapshot %s(%s)(%s) with xattrs %+v", snapshotName, snapSvcLvol.Alias, snapSvcLvol.UUID, xattrs)

	return ServiceReplicaToProtoReplica(r), err
}

func (r *Replica) SnapshotDelete(spdkClient *spdkclient.Client, snapshotName string) (replica *spdkrpc.Replica, err error) {
	updateRequired := false

	r.Lock()
	defer func() {
		r.Unlock()

		if updateRequired {
			r.UpdateCh <- nil
		}
	}()

	if r.State != types.InstanceStateStopped && r.State != types.InstanceStateRunning {
		return nil, fmt.Errorf("invalid state %v for replica %s snapshot deletion", r.State, r.Name)
	}

	snapLvolName := GetReplicaSnapshotLvolName(r.Name, snapshotName)
	snapSvcLvol := r.SnapshotLvolMap[snapLvolName]
	if snapSvcLvol == nil {
		return ServiceReplicaToProtoReplica(r), nil
	}
	if len(snapSvcLvol.Children) > 1 {
		return nil, fmt.Errorf("cannot delete snapshot %s(%s) since it has %d children", snapshotName, snapLvolName, len(snapSvcLvol.Children))
	}

	defer func() {
		if err != nil {
			if r.State != types.InstanceStateError {
				r.State = types.InstanceStateError
				updateRequired = true
			}
			r.ErrorMsg = err.Error()
			r.log.WithError(err).Errorf("Failed to delete snapshot %s for replica %s", snapshotName, r.Name)
		} else {
			if r.State != types.InstanceStateError {
				r.ErrorMsg = ""
			}
		}
	}()

	if err := r.stopSnapshotHash(spdkClient, snapSvcLvol); err != nil {
		return nil, errors.Wrapf(err, "failed to stop snapshot %s(%s) checksum hashing before snapshot deletion", snapLvolName, snapshotName)
	}

	if _, err := spdkClient.BdevLvolDelete(snapSvcLvol.UUID); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return nil, err
	}
	r.removeLvolFromSnapshotLvolMapWithoutLock(snapLvolName)
	r.removeLvolFromActiveChainWithoutLock(snapLvolName)
	for _, childSvcLvol := range snapSvcLvol.Children {
		bdevLvol, err := spdkClient.BdevLvolGetByName(childSvcLvol.UUID, 0)
		if err != nil {
			return nil, err
		}
		if err := r.stopSnapshotHash(spdkClient, childSvcLvol); err != nil {
			return nil, errors.Wrapf(err, "failed to stop child snapshot %s checksum hashing after snapshot %s deletion", childSvcLvol.Name, snapshotName)
		}
		childSvcLvol.ActualSize = bdevLvol.DriverSpecific.Lvol.NumAllocatedClusters * defaultClusterSize
		childSvcLvol.SnapshotChecksum = ""
	}

	updateRequired = true

	r.log.Infof("Replica deleted snapshot %s(%s)(%s)", snapshotName, snapSvcLvol.Alias, snapSvcLvol.UUID)

	return ServiceReplicaToProtoReplica(r), nil
}

func (r *Replica) removeLvolFromSnapshotLvolMapWithoutLock(snapsLvolName string) {
	var deletingSvcLvol, parentSvcLvol, childSvcLvol *Lvol

	deletingSvcLvol = r.SnapshotLvolMap[snapsLvolName]
	if IsReplicaSnapshotLvol(r.Name, deletingSvcLvol.Parent) {
		parentSvcLvol = r.SnapshotLvolMap[deletingSvcLvol.Parent]
	} else {
		// Parent is either backing image or nil
		parentSvcLvol = r.ActiveChain[0]
	}
	if parentSvcLvol != nil {
		delete(parentSvcLvol.Children, deletingSvcLvol.Name)
	}
	for _, childSvcLvol = range deletingSvcLvol.Children {
		if parentSvcLvol != nil {
			parentSvcLvol.Children[childSvcLvol.Name] = childSvcLvol
			childSvcLvol.Parent = parentSvcLvol.Name
		} else {
			childSvcLvol.Parent = ""
		}
	}

	delete(r.SnapshotLvolMap, snapsLvolName)
}

func (r *Replica) removeLvolFromActiveChainWithoutLock(snapLvolName string) int {
	pos := -1
	for idx, lvol := range r.ActiveChain {
		// Cannot remove the backing image from the chain
		if idx == 0 {
			continue
		}
		if lvol.Name == snapLvolName {
			pos = idx
			break
		}
	}

	// Cannot remove backing image lvol or head lvol
	prevChain := r.ActiveChain
	if pos >= 1 && pos < len(r.ActiveChain)-1 {
		r.ActiveChain = append([]*Lvol{}, prevChain[:pos]...)
		r.ActiveChain = append(r.ActiveChain, prevChain[pos+1:]...)
	}

	return pos
}

func (r *Replica) SnapshotRevert(spdkClient *spdkclient.Client, snapshotName string) (pReplica *spdkrpc.Replica, err error) {
	updateRequired := false

	r.Lock()
	defer func() {
		r.Unlock()

		if updateRequired {
			r.UpdateCh <- nil
		}
	}()

	if r.State != types.InstanceStateStopped && r.State != types.InstanceStateRunning {
		return nil, fmt.Errorf("invalid state %v for replica %s snapshot revert", r.State, r.Name)
	}

	snapLvolName := GetReplicaSnapshotLvolName(r.Name, snapshotName)
	snapSvcLvol := r.SnapshotLvolMap[snapLvolName]
	if snapSvcLvol == nil {
		return nil, fmt.Errorf("cannot revert to a non-existing snapshot %s(%s)", snapshotName, snapLvolName)
	}

	defer func() {
		if err != nil && r.State != types.InstanceStateError {
			r.State = types.InstanceStateError
			updateRequired = true
		}
	}()

	if len(r.ActiveChain) < 2 {
		return nil, fmt.Errorf("invalid chain length %d for replica snapshot revert", len(r.ActiveChain))
	}

	if _, err := spdkClient.BdevLvolDelete(r.Alias); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return nil, err
	}
	// The parent of the old head lvol is a valid snapshot lvol or backing image lvol
	if r.ActiveChain[len(r.ActiveChain)-2] != nil {
		delete(r.ActiveChain[len(r.ActiveChain)-2].Children, r.Name)
	}
	r.Head = nil
	r.ActiveChain = r.ActiveChain[:len(r.ActiveChain)-1]

	// TODO: If the below steps fail, there will be no head lvol for the replica. Need to guarantee that the replica can be cleaned up correctly in this case

	if err := r.stopSnapshotHash(spdkClient, snapSvcLvol); err != nil {
		return nil, errors.Wrapf(err, "failed to stop snapshot %s(%s) checksum hashing before snapshot revert", snapLvolName, snapshotName)
	}

	headLvolUUID, err := spdkClient.BdevLvolClone(snapSvcLvol.UUID, r.Name)
	if err != nil {
		return nil, err
	}

	bdevLvolMap, err := GetBdevLvolMapWithFilter(spdkClient, r.replicaLvolFilter)
	if err != nil {
		return nil, err
	}

	newSnapshotLvolMap, err := constructSnapshotLvolMap(r.Name, bdevLvolMap)
	if err != nil {
		return nil, err
	}
	newChain, err := constructActiveChainFromSnapshotLvolMap(r.Name, newSnapshotLvolMap, bdevLvolMap)
	if err != nil {
		return nil, err
	}

	r.Head = newChain[len(newChain)-1]
	r.ActiveChain = newChain
	r.SnapshotLvolMap = newSnapshotLvolMap

	if r.IsExposed {
		r.log.Infof("Unexposing bdev before reverting snapshot %s for replica %s", snapshotName, r.Name)
		if err := spdkClient.StopExposeBdev(helpertypes.GetNQN(r.Name)); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return nil, err
		}
		r.IsExposed = false

		if err := spdkClient.StartExposeBdev(helpertypes.GetNQN(r.Name), headLvolUUID, generateNGUID(r.Name), r.IP, strconv.Itoa(int(r.PortStart))); err != nil {
			return nil, err
		}
		r.IsExposed = true
	}

	updateRequired = true

	r.log.Infof("Replica reverted snapshot %s(%s)(%s)", snapshotName, snapSvcLvol.Alias, snapSvcLvol.UUID)

	return ServiceReplicaToProtoReplica(r), nil
}

// SnapshotPurge asks the replica to delete all system created snapshots
func (r *Replica) SnapshotPurge(spdkClient *spdkclient.Client) (err error) {
	updateRequired := false

	r.Lock()
	defer func() {
		r.Unlock()

		if updateRequired {
			r.UpdateCh <- nil
		}
	}()

	defer func() {
		if err != nil && r.State != types.InstanceStateError {
			r.State = types.InstanceStateError
			updateRequired = true
		}
	}()

	if len(r.ActiveChain) < 2 {
		return fmt.Errorf("invalid chain length %d for replica snapshot purge", len(r.ActiveChain))
	}

	// delete all non-user-created snapshots
	var purgeList []string
	for snapshotLvolName, snapSvcLvol := range r.SnapshotLvolMap {
		logrus.Infof("Considering snapshot lvol %s for purge: %+v", snapshotLvolName, snapSvcLvol)
		if snapSvcLvol.UserCreated {
			logrus.Infof("Skipping user created snapshot lvol %s for purge", snapshotLvolName)
			continue
		}
		if len(snapSvcLvol.Children) > 1 {
			logrus.Infof("Skipping snapshot lvol %s for purge since it has %d children", snapshotLvolName, len(snapSvcLvol.Children))
			continue
		}

		if err := r.stopSnapshotHash(spdkClient, snapSvcLvol); err != nil {
			logrus.Infof("Failed to stop snapshot lvol %s checksum hashing before snapshot purge: %v", snapshotLvolName, err)
			return errors.Wrapf(err, "failed to stop snapshot lvol %s checksum hashing before snapshot purge", snapshotLvolName)
		}
		if _, err := spdkClient.BdevLvolDelete(snapSvcLvol.UUID); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return errors.Wrapf(err, "failed to delete snapshot lvol %s before snapshot purge", snapshotLvolName)
		}
		purgeList = append(purgeList, snapshotLvolName)

		for _, childSvcLvol := range snapSvcLvol.Children {
			if err := r.stopSnapshotHash(spdkClient, childSvcLvol); err != nil {
				return errors.Wrapf(err, "failed to stop child snapshot lvol %s checksum hashing after snapshot lvol %s purge", childSvcLvol.Name, snapshotLvolName)
			}
			childSvcLvol.SnapshotChecksum = ""
		}

		r.removeLvolFromSnapshotLvolMapWithoutLock(snapshotLvolName)
		r.removeLvolFromActiveChainWithoutLock(snapshotLvolName)

		for _, childSvcLvol := range snapSvcLvol.Children {
			bdevLvol, err := spdkClient.BdevLvolGetByName(childSvcLvol.UUID, 0)
			if err != nil {
				return errors.Wrapf(err, "failed to get child snapshot lvol %s after snapshot lvol %s purge", childSvcLvol.Name, snapshotLvolName)
			}
			childSvcLvol.ActualSize = bdevLvol.DriverSpecific.Lvol.NumAllocatedClusters * defaultClusterSize
		}

		updateRequired = true
	}

	r.log.Infof("Replica purged system created snapshot lvols: %+v", purgeList)

	return nil
}

// SnapshotHash asks the replica to calculate/hash checksum for a snapshot lvol
func (r *Replica) SnapshotHash(spdkClient *spdkclient.Client, snapshotName string, rehash bool) (err error) {
	updateRequired := false

	r.Lock()
	defer func() {
		r.Unlock()

		if updateRequired {
			r.UpdateCh <- nil
		}
	}()

	defer func() {
		if err != nil {
			r.log.Warnf("Replica failed to hash checksum for snapshot %s: %v", snapshotName, err)
		}
	}()

	if len(r.ActiveChain) < 2 {
		r.State = types.InstanceStateError
		updateRequired = true
		return fmt.Errorf("invalid chain length %d for replica snapshot purge", len(r.ActiveChain))
	}

	snapLvolName := GetReplicaSnapshotLvolName(r.Name, snapshotName)
	snapSvcLvol := r.SnapshotLvolMap[snapLvolName]
	if snapSvcLvol == nil {
		return fmt.Errorf("cannot find snapshot %s(%s) for replica %s snapshot hash", snapshotName, snapLvolName, r.Name)
	}
	if !rehash && snapSvcLvol.SnapshotChecksum != "" {
		return nil
	}

	snapParentSvcLvol := r.SnapshotLvolMap[snapSvcLvol.Parent]
	if !snapSvcLvol.UserCreated || (snapParentSvcLvol != nil && !snapParentSvcLvol.UserCreated) {
		if r.isRebuilding || r.rebuildingSrcCache.dstReplicaName != "" {
			return fmt.Errorf("cannot hash snapshot %s(%s)(%s) checksum, since its parent or itself is a system created snapshot while the replica is rebuilding", snapshotName, snapLvolName, snapSvcLvol.UUID)
		}
	}

	hashStatusValue, exists := r.SnapshotLvolHashStatusMap.Load(snapLvolName)
	hashStatus, ok := hashStatusValue.(LvolHashStatus)
	if exists && ok {
		if hashStatus.State == types.ProgressStateInProgress {
			return fmt.Errorf("replica %s range hash is in progress, cannot do it for snapshot %s(%s)(%s)", r.Name, snapshotName, snapLvolName, snapSvcLvol.UUID)
		}
		if hashStatus.State == types.ProgressStateError {
			r.log.Infof("Replica is restarting range hash for snapshot %s(%s)(%s), previous hash error: %s", snapshotName, snapLvolName, snapSvcLvol.UUID, hashStatus.Error)
		}
		// TODO: If we need to handle `hashStatus.State == types.ProgressStateComplete` when `snapSvcLvol.SnapshotChecksum == ""`
	}

	r.log.Debugf("Replica is hashing range checksum for snapshot %s(%s)(%s)", snapshotName, snapLvolName, snapSvcLvol.UUID)
	hashStatus = LvolHashStatus{
		State: types.ProgressStateInProgress,
	}
	if rehash {
		hashStatus.PreviousChecksum = snapSvcLvol.SnapshotChecksum
	}
	r.SnapshotLvolHashStatusMap.Store(snapLvolName, hashStatus)

	go func() {
		_, err := spdkClient.BdevLvolRegisterRangeChecksums(snapSvcLvol.Alias)
		if err != nil {
			hashStatus.State = types.ProgressStateError
			hashStatus.Error = err.Error()
			r.SnapshotLvolHashStatusMap.Store(snapLvolName, hashStatus)
			r.log.WithError(err).Errorf("Replica failed to hash range checksum for snapshot %s (%s) (%s)", snapshotName, snapLvolName, snapSvcLvol.UUID)
			return
		}
		r.log.Infof("Replica completed to hash range checksum for snapshot %s (%s) (%s)", snapshotName, snapLvolName, snapSvcLvol.UUID)
	}()

	return nil
}

// SnapshotHashStatus asks the replica snapshot lvol checksum status
func (r *Replica) SnapshotHashStatus(snapshotName string) (state, checksum, errMsg string, silentlyCorrupted bool, err error) {
	r.Lock()
	defer func() {
		r.Unlock()

		if err != nil && r.State != types.InstanceStateError {
			r.log.WithError(err).Warnf("Replica %v failed to get hash status for snapshot %s", r.Name, snapshotName)
		}
	}()

	if len(r.ActiveChain) < 2 {
		return "", "", "", false, fmt.Errorf("invalid chain length %d for replica snapshot hash status", len(r.ActiveChain))
	}

	snapLvolName := GetReplicaSnapshotLvolName(r.Name, snapshotName)
	snapSvcLvol := r.SnapshotLvolMap[snapLvolName]
	if snapSvcLvol == nil {
		return "", "", "", false, fmt.Errorf("cannot find snapshot %s(%s) for replica %s snapshot hash status", snapshotName, snapLvolName, r.Name)
	}
	r.SyncSnapshotHashStatus(snapSvcLvol)

	hashStatusValue, ok := r.SnapshotLvolHashStatusMap.Load(snapLvolName)
	if !ok {
		return "", "", "", false, nil
	}

	hashStatus := hashStatusValue.(LvolHashStatus)
	// TODO: For now we will try to find a better way to detect silently corrupted snapshots rather than relying on hashStatus.PreviousChecksum.
	//silentlyCorrupted = hashStatus.PreviousChecksum != "" && hashStatus.Checksum != "" && hashStatus.PreviousChecksum != hashStatus.Checksum
	return hashStatus.State, hashStatus.Checksum, hashStatus.Error, silentlyCorrupted, nil
}

// SnapshotRangeHashGet asks the replica snapshot lvol get the checksums for a specific range of clusters
func (r *Replica) SnapshotRangeHashGet(spdkClient *spdkclient.Client, snapshotName string, clusterStartIndex, clusterCount uint64) (rangeHashMap map[uint64]uint64, err error) {
	r.Lock()
	defer func() {
		r.Unlock()

		if err != nil && r.State != types.InstanceStateError {
			r.log.WithError(err).Warnf("Replica failed to get snapshot %s range [%d, %d) hash map", snapshotName, clusterStartIndex, clusterStartIndex+clusterCount)
		}
	}()

	if len(r.ActiveChain) < 2 {
		return nil, fmt.Errorf("invalid chain length %d for replica %s snapshot %s range [%d, %d) hash map get", len(r.ActiveChain), r.Name, snapshotName, clusterStartIndex, clusterStartIndex+clusterCount)
	}

	snapLvolName := GetReplicaSnapshotLvolName(r.Name, snapshotName)
	snapSvcLvol := r.SnapshotLvolMap[snapLvolName]
	if snapSvcLvol == nil {
		return nil, fmt.Errorf("cannot find snapshot %s(%s) for replica %s snapshot range [%d, %d) hash map get", snapshotName, snapLvolName, r.Name, clusterStartIndex, clusterStartIndex+clusterCount)
	}

	return spdkClient.BdevLvolGetRangeChecksums(spdktypes.GetLvolAlias(r.LvsName, snapLvolName), clusterStartIndex, clusterCount)
}

// SnapshotCloneDstStart asks the destination replica to start snapshot cloning
func (r *Replica) SnapshotCloneDstStart(spdkClient *spdkclient.Client, snapshotName, srcReplicaName, srcReplicaAddress string, cloneMode spdkrpc.CloneMode) (err error) {
	updateRequired := false

	r.Lock()
	defer func() {
		r.Unlock()

		if updateRequired {
			r.UpdateCh <- nil
		}
	}()

	if r.isSnapshotCloning {
		return fmt.Errorf("replica %s cloning is in process", r.Name)
	}
	r.isSnapshotCloning = true

	defer func() {
		if err != nil {
			r.log.WithError(err).Errorf("Clone dst replica failed to do SnapshotCloneDstStart for snapshot %v with "+
				"srcReplicaName %v, srcReplicaAddress %v", snapshotName, srcReplicaName, srcReplicaAddress)
			if r.State != types.InstanceStateError {
				r.State = types.InstanceStateError
			}
			r.ErrorMsg = err.Error()
			if r.snapshotCloningDstCache.cloningError == "" {
				r.snapshotCloningDstCache.cloningError = err.Error()
				r.snapshotCloningDstCache.cloningState = types.ProgressStateError
			}
		} else {
			if r.State != types.InstanceStateError {
				r.ErrorMsg = ""
			}
		}

		updateRequired = true
	}()

	// Replica.Delete and Replica.Create do not guarantee that the previous cloning dst replica info is cleaned up
	if err := r.doCleanupForSnapshotCloneDst(spdkClient, true); err != nil {
		return errors.Wrapf(err, "failed to clean up the previous cloning dst info for dst replica snapshot "+
			"clone start, src replica name %s, address %s, snapshot name %s", r.snapshotCloningDstCache.srcReplicaName,
			r.snapshotCloningDstCache.srcReplicaAddress, r.snapshotCloningDstCache.snapshotName)
	}
	// init cloning
	r.snapshotCloningDstCache.snapshotName = snapshotName
	r.snapshotCloningDstCache.srcReplicaName = srcReplicaName
	r.snapshotCloningDstCache.srcReplicaAddress = srcReplicaAddress

	if cloneMode == spdkrpc.CloneMode_CLONE_MODE_LINKED_CLONE {
		srcReplicaIP, _, err := splitHostPort(srcReplicaAddress)
		if err != nil {
			return errors.Wrapf(err, "failed to split src Replica address %v", srcReplicaAddress)
		}

		if r.IP != srcReplicaIP {
			return fmt.Errorf("failed to do snapshot linked-clone: dst replica IP %v is not the same as "+
				"src replica IP %v", r.IP, srcReplicaIP)
		}

		srcReplicaServiceCli, err := GetServiceClient(r.snapshotCloningDstCache.srcReplicaAddress)
		if err != nil {
			return err
		}
		defer func() {
			if errClose := srcReplicaServiceCli.Close(); errClose != nil {
				r.log.WithError(errClose).Errorf("Failed to close replica %s client with address %s during "+
					"start cloning at dst", r.snapshotCloningDstCache.srcReplicaName, r.snapshotCloningDstCache.srcReplicaAddress)
			}
		}()

		if err := srcReplicaServiceCli.ReplicaSnapshotCloneSrcStart(r.snapshotCloningDstCache.srcReplicaName,
			snapshotName, r.Name, "", cloneMode); err != nil {
			return err
		}
		r.log.Infof("Clone dst replica updated clone state from %v to %v", r.snapshotCloningDstCache.cloningState, types.ProgressStateComplete)
		r.snapshotCloningDstCache.cloningState = types.ProgressStateComplete
		return r.SnapshotCloneDstFinish(spdkClient, cloneMode)
	}

	if r.snapshotCloningDstCache.cloningPort == 0 {
		if r.snapshotCloningDstCache.cloningPort, _, err = r.portAllocator.AllocateRange(1); err != nil {
			return errors.Wrapf(err, "failed to allocate a cloning port for dst replica %v snapshot clone start", r.Name)
		}
	}
	// Create cloning lvol and expose it
	cloningLvolName := GetReplicaCloningLvolName(r.Name)
	if _, err = spdkClient.BdevLvolCreate("", r.LvsUUID, cloningLvolName, util.BytesToMiB(r.SpecSize),
		"", true); err != nil {
		return err
	}
	cloningLvolAlias := spdktypes.GetLvolAlias(r.LvsName, cloningLvolName)
	cloningBdevLvol, err := spdkClient.BdevLvolGetByName(cloningLvolAlias, 0)
	if err != nil {
		return err
	}
	r.snapshotCloningDstCache.cloningLvol = BdevLvolInfoToServiceLvol(&cloningBdevLvol)

	if err := spdkClient.StartExposeBdev(helpertypes.GetNQN(r.snapshotCloningDstCache.cloningLvol.Name),
		r.snapshotCloningDstCache.cloningLvol.UUID, generateNGUID(r.snapshotCloningDstCache.cloningLvol.Name), r.IP,
		strconv.Itoa(int(r.snapshotCloningDstCache.cloningPort))); err != nil {
		return err
	}
	dstCloningLvolAddress := net.JoinHostPort(r.IP, strconv.Itoa(int(r.snapshotCloningDstCache.cloningPort)))

	// Ask src replica to start cloning
	srcReplicaServiceCli, err := GetServiceClient(r.snapshotCloningDstCache.srcReplicaAddress)
	if err != nil {
		return err
	}
	defer func() {
		if errClose := srcReplicaServiceCli.Close(); errClose != nil {
			r.log.WithError(errClose).Errorf("Clone dst replica with address %s failed to close src replica %s client during clone start",
				r.snapshotCloningDstCache.srcReplicaAddress, r.snapshotCloningDstCache.srcReplicaName)
		}
	}()

	if err := srcReplicaServiceCli.ReplicaSnapshotCloneSrcStart(r.snapshotCloningDstCache.srcReplicaName, snapshotName,
		r.Name, dstCloningLvolAddress, cloneMode); err != nil {
		return err
	}
	r.snapshotCloningDstCache.cloningState = types.ProgressStateInProgress

	monitorCtx, monitorCancelFunc := context.WithTimeout(context.Background(), MaxSnapshotCloneWaitTime)
	r.snapshotCloningDstCache.monitorCancelFunc = monitorCancelFunc

	r.log.Infof("Clone dst replica sent a snapshot %s clone request to src replica %v at address %v", snapshotName, srcReplicaName, srcReplicaAddress)

	go r.monitorSnapshotClone(spdkClient, monitorCtx, monitorCancelFunc, srcReplicaName, srcReplicaAddress, snapshotName, cloneMode)

	return nil
}

func (r *Replica) monitorSnapshotClone(spdkCli *spdkclient.Client, ctx context.Context, cancel context.CancelFunc,
	srcReplicaName, srcReplicaAddress, snapshotName string, cloneMode spdkrpc.CloneMode) {

	ticker := time.NewTicker(SnapshotCloneStatusCheckInterval)
	defer func() {
		ticker.Stop()
		// Best-effort: tell src to finish.
		if srcReplicaCli, err := GetServiceClient(srcReplicaAddress); err != nil {
			r.log.WithError(err).Errorf("Clone dst replica failed to create src replica %s client to finish snapshot %s cloning", srcReplicaName, snapshotName)
		} else {
			if err := srcReplicaCli.ReplicaSnapshotCloneSrcFinish(srcReplicaName, r.Name); err != nil {
				r.log.WithError(err).Errorf("Clone dst replica failed to tell src replica %s to finish snapshot %s cloning", srcReplicaName, snapshotName)
			}
			if err := srcReplicaCli.Close(); err != nil {
				r.log.WithError(err).Errorf("Clone dst replica failed to close src replica %s client after finish for snapshot %s", srcReplicaName, snapshotName)
			}
		}

		if err := r.SnapshotCloneDstFinish(spdkCli, cloneMode); err != nil {
			r.log.WithError(err).Errorf("Clone dst replica failed to finish snapshot %s cloning", snapshotName)
		}

		if cancel != nil {
			cancel()
		}
	}()

	setStatus := func(state string, msg string, progress ...uint64) {
		r.Lock()
		defer r.Unlock()
		if !r.isSnapshotCloning {
			return
		}
		r.snapshotCloningDstCache.cloningState = state
		r.snapshotCloningDstCache.cloningError = msg
		if len(progress) == 2 { // only touch if provided
			r.snapshotCloningDstCache.processedClusters = progress[0]
			r.snapshotCloningDstCache.totalClusters = progress[1]
		}
	}

	retries := 0
	for {
		select {
		case <-ctx.Done():
			var reason string
			if errors.Is(ctx.Err(), context.Canceled) {
				reason = "operation is aborted"
			} else if errors.Is(ctx.Err(), context.DeadlineExceeded) {
				reason = "operation is timed out"
			} else {
				reason = ctx.Err().Error()
			}
			r.log.Warnf("Clone dst replica failed ReplicaSnapshotCloneSrcStatusCheck, reason: %s", reason)
			setStatus(types.ProgressStateError, "failed to check ReplicaSnapshotCloneSrcStatusCheck: "+reason)
			return
		case <-ticker.C:
			srcReplicaCli, err := GetServiceClient(srcReplicaAddress)
			if err != nil {
				retries++
				if retries > maxRetries {
					msg := fmt.Sprintf("Clone dst replica %s failed to create src replica %s client over %d times. Setting cloning to error", r.Name, srcReplicaName, retries)
					r.log.WithError(err).Error(msg)
					setStatus(types.ProgressStateError, msg)
					return
				}
				r.log.WithError(err).Warnf("Clone dst replica failed to create src client for %s (retry %d)", srcReplicaName, retries)
				continue
			}
			status, err := srcReplicaCli.ReplicaSnapshotCloneSrcStatusCheck(srcReplicaName, snapshotName, r.Name)
			if errClose := srcReplicaCli.Close(); errClose != nil {
				r.log.WithError(errClose).Errorf("Clone dst replica failed to close src client for %s after status check", srcReplicaName)
			}
			if err != nil {
				retries++
				if retries > maxRetries {
					msg := fmt.Sprintf(
						"Clone dst Replica failed to check snapshot clone status from src replica %s for snapshot %s over %d times. Setting snapshot cloning to error", srcReplicaName, snapshotName, retries,
					)
					r.log.WithError(err).Error(msg)
					setStatus(types.ProgressStateError, msg)
					return
				}
				r.log.WithError(err).Warnf("Clone dst Replica failed to check snapshot clone status from src replica %v for snapshot %v (retry %v)", srcReplicaName, snapshotName, retries)
				continue
			}
			retries = 0

			setStatus(status.State, status.ErrorMsg, status.ProcessedClusters, status.TotalClusters)

			if status.State == types.ProgressStateError || status.State == types.ProgressStateComplete {
				r.log.Infof("Clone dst replica stopped to monitor snapshot %s clone as the state is updated to %v", snapshotName, status.State)
				return
			}
		}
	}
}

func (r *Replica) SnapshotCloneDstStatusCheck() (status *spdkrpc.ReplicaSnapshotCloneDstStatusCheckResponse, err error) {
	r.Lock()
	defer r.Unlock()

	defer func() {
		err = errors.Wrapf(err, "failed to check snapshot clone status in dst replica %v", r.Name)
	}()

	c := r.snapshotCloningDstCache
	var progress uint32
	switch {
	case c.totalClusters == 0:
		progress = 0
	case c.processedClusters >= c.totalClusters || c.cloningState == types.ProgressStateComplete:
		progress = 100
	default:
		pct := math.Ceil((float64(c.processedClusters) / float64(c.totalClusters)) * 100)
		if pct > 100 { // guard against float quirks and >100%
			pct = 100
		}
		progress = uint32(pct)
	}

	return &spdkrpc.ReplicaSnapshotCloneDstStatusCheckResponse{
		IsCloning:         r.isSnapshotCloning,
		SrcReplicaName:    c.srcReplicaName,
		SrcReplicaAddress: c.srcReplicaAddress,
		SnapshotName:      c.snapshotName,
		State:             c.cloningState,
		Progress:          progress,
		Error:             c.cloningError,
	}, nil
}

func (r *Replica) SnapshotCloneDstFinish(spdkClient *spdkclient.Client, cloneMode spdkrpc.CloneMode) (err error) {
	if cloneMode == spdkrpc.CloneMode_CLONE_MODE_LINKED_CLONE {
		r.isSnapshotCloning = false
		return nil
	}

	updateRequired := false

	r.Lock()
	defer func() {
		r.Unlock()

		if updateRequired {
			r.UpdateCh <- nil
		}
	}()

	if !r.isSnapshotCloning {
		return fmt.Errorf("replica %s is not in cloning", r.Name)
	}

	defer func() {
		if err != nil {
			if r.State != types.InstanceStateError {
				r.State = types.InstanceStateError
			}
			r.ErrorMsg = err.Error()
			if r.snapshotCloningDstCache.cloningError == "" {
				r.snapshotCloningDstCache.cloningError = err.Error()
				r.snapshotCloningDstCache.cloningState = types.ProgressStateError
			}
		} else {
			if r.State != types.InstanceStateError {
				r.ErrorMsg = ""
			}
		}

		updateRequired = true
	}()

	if r.snapshotCloningDstCache.cloningState == types.ProgressStateComplete {
		if r.Head == nil {
			return fmt.Errorf("cannot find the head for replica %s snapshot clone finish", r.Name)
		}
		if r.snapshotCloningDstCache.cloningLvol == nil {
			return fmt.Errorf("cannot find the head for cloning lvol for snapshot clone finish in replica %v", r.Name)
		}
		tmpSnapName := GetTmpSnapNameForCloningLvol(r.Name)
		snapUUID, err := spdkClient.BdevLvolSnapshot(r.snapshotCloningDstCache.cloningLvol.UUID, tmpSnapName, []spdkclient.Xattr{})
		if err != nil {
			return err
		}
		snapBdevLvol, err := spdkClient.BdevLvolGetByName(snapUUID, 0)
		if err != nil {
			return err
		}
		tmpSnap := BdevLvolInfoToServiceLvol(&snapBdevLvol)
		if _, err := spdkClient.BdevLvolSetParent(r.Head.Alias, tmpSnap.Alias); err != nil {
			return err
		}
	}

	if err = r.doCleanupForSnapshotCloneDst(spdkClient, false); err != nil {
		return err
	}

	r.isSnapshotCloning = false

	return
}

// doCleanupForSnapshotCloneDst blindly cleans up the dst replica cloning cache and all redundant lvols if any
func (r *Replica) doCleanupForSnapshotCloneDst(spdkClient *spdkclient.Client, clearStatus bool) error {
	aggregatedErrors := []error{}

	// Blindly clean up the cloning lvol and the exposed port
	cloningLvolName := GetReplicaCloningLvolName(r.Name)
	if r.snapshotCloningDstCache.cloningLvol != nil && r.snapshotCloningDstCache.cloningLvol.Name != cloningLvolName {
		err := fmt.Errorf("BUG: replica %s cloning lvol actual name %s does not match the expected name %v, will use the actual name for the cleanup", r.Name, r.snapshotCloningDstCache.cloningLvol.Name, cloningLvolName)
		r.log.Error(err)
		aggregatedErrors = append(aggregatedErrors, err)
		cloningLvolName = r.snapshotCloningDstCache.cloningLvol.Name
	}
	r.log.Infof("Cleaning up cloning lvol %s and the exposed port %d for snapshot clone dst replica %s", cloningLvolName, r.snapshotCloningDstCache.cloningPort, r.Name)
	if err := spdkClient.StopExposeBdev(helpertypes.GetNQN(cloningLvolName)); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		r.log.WithError(err).Errorf("Failed to stop exposing the cloning lvol %s for cloning dst cleanup", cloningLvolName)
		aggregatedErrors = append(aggregatedErrors, err)
	}
	if r.snapshotCloningDstCache.cloningPort != 0 {
		if err := r.portAllocator.ReleaseRange(r.snapshotCloningDstCache.cloningPort, r.snapshotCloningDstCache.cloningPort); err != nil {
			r.log.WithError(err).Errorf("Failed to release the cloning port %d for cloning dst cleanup", r.snapshotCloningDstCache.cloningPort)
			aggregatedErrors = append(aggregatedErrors, err)
		} else {
			r.snapshotCloningDstCache.cloningPort = 0
			r.snapshotCloningDstCache.cloningLvolAddress = ""
		}
	}
	if _, err := spdkClient.BdevLvolDelete(spdktypes.GetLvolAlias(r.LvsName, cloningLvolName)); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		r.log.WithError(err).Errorf("Failed to delete the cloning lvol %s for cloning dst cleanup", cloningLvolName)
		aggregatedErrors = append(aggregatedErrors, err)
	} else {
		r.snapshotCloningDstCache.cloningLvol = nil
	}

	tmpSnapName := GetTmpSnapNameForCloningLvol(r.Name)
	if _, err := spdkClient.BdevLvolDelete(spdktypes.GetLvolAlias(r.LvsName, tmpSnapName)); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		r.log.WithError(err).Errorf("Failed to delete the tmp snapshot %s for cloning dst cleanup", tmpSnapName)
		aggregatedErrors = append(aggregatedErrors, err)
	}

	r.snapshotCloningDstCache.srcReplicaName = ""
	r.snapshotCloningDstCache.srcReplicaAddress = ""
	if r.snapshotCloningDstCache.monitorCancelFunc != nil {
		r.snapshotCloningDstCache.monitorCancelFunc()
		r.snapshotCloningDstCache.monitorCancelFunc = nil
	}

	if clearStatus {
		r.snapshotCloningDstCache.processedClusters = 0
		r.snapshotCloningDstCache.totalClusters = 0
		r.snapshotCloningDstCache.cloningError = ""
		r.snapshotCloningDstCache.cloningState = ""
	}

	return util.CombineErrors(aggregatedErrors...)
}

func (r *Replica) snapshotLinkedCloneSrcStart(spdkClient *spdkclient.Client, snapshotName, dstReplicaName string) (err error) {
	defer func() {
		err = errors.Wrap(err, "failed to do snapshotLinkedCloneSrcStart")
	}()

	bdevLvolMap, err := GetBdevLvolMapWithFilter(spdkClient, r.replicaLvolFilter)
	if err != nil {
		return err
	}

	existingParentOfDstReplica := ""
	for lvolName, lvol := range bdevLvolMap {
		if types.IsBackingImageSnapLvolName(lvolName) {
			continue
		}
		for _, childLvolName := range lvol.DriverSpecific.Lvol.Clones {
			if childLvolName == dstReplicaName {
				existingParentOfDstReplica = lvolName
				continue
			}
			if !IsReplicaLvol(r.Name, childLvolName) {
				return fmt.Errorf("there are already another linked-clone lvol %v in src replica %v. "+
					"Each src replica can only has 1 linked-clone lvol at a time", childLvolName, r.Name)
			}
		}
	}

	snapLvolName := GetReplicaSnapshotLvolName(r.Name, snapshotName)
	snapLvol := r.SnapshotLvolMap[snapLvolName]
	if snapLvol == nil {
		return fmt.Errorf("cannot find snapshot %s for src replica %s", snapshotName, r.Name)
	}

	if existingParentOfDstReplica != "" {
		if existingParentOfDstReplica != snapLvolName {
			return fmt.Errorf("dst replica already has a different parent %q than the snapshot %v", dstReplicaName, snapshotName)
		}
		// Operation is already satisfied
		return nil
	}

	set, err := spdkClient.BdevLvolSetParent(spdktypes.GetLvolAlias(r.LvsName, dstReplicaName), snapLvol.Alias)
	if err != nil {
		return err
	}
	if !set {
		return fmt.Errorf("failed set lvol %v as the parent of %v", snapLvol.Alias, dstReplicaName)
	}

	return nil
}

// SnapshotCloneSrcStart asks the src replica to start snapshot cloning
func (r *Replica) SnapshotCloneSrcStart(spdkClient *spdkclient.Client, snapshotName, dstReplicaName, dstCloningLvolAddress string, cloneMode spdkrpc.CloneMode) (err error) {
	r.Lock()
	defer r.Unlock()

	if c := r.snapshotCloningSrcCache[dstReplicaName]; c != nil {
		if err := doCleanupForSnapshotCloneSrc(spdkClient, c); err != nil {
			return err
		}
	}
	c := &SnapshotCloningSrcCache{
		dstReplicaName: dstReplicaName,
		snapshotName:   snapshotName,
	}
	r.snapshotCloningSrcCache[dstReplicaName] = c

	r.log.Infof("Clone src relica is starting snapshot %s clone for dst replica %v with cloning lvol address %v", snapshotName, dstReplicaName, dstCloningLvolAddress)

	if cloneMode == spdkrpc.CloneMode_CLONE_MODE_LINKED_CLONE {
		return r.snapshotLinkedCloneSrcStart(spdkClient, snapshotName, dstReplicaName)
	}

	snapLvolName := GetReplicaSnapshotLvolName(r.Name, snapshotName)
	snapLvol := r.SnapshotLvolMap[snapLvolName]
	if snapLvol == nil {
		return fmt.Errorf("cannot find snapshot %s for src replica %s for SnapshotCloneSrcStart", snapshotName, r.Name)
	}

	dstCloningLvolName := GetReplicaCloningLvolName(dstReplicaName)
	dstCloningBdevName, err := connectNVMfBdev(spdkClient, dstCloningLvolName, dstCloningLvolAddress,
		replicaCtrlrLossTimeoutSec, replicaFastIOFailTimeoutSec, maxRetries, retryInterval)
	if err != nil {
		return err
	}
	c.dstCloningBdevName = dstCloningBdevName

	opID, err := spdkClient.BdevLvolStartDeepCopy(snapLvol.UUID, c.dstCloningBdevName)
	if err != nil {
		return err
	}
	c.deepCopyOpID = opID
	c.deepCopyStatus = DeepCopyStatus{}
	return nil
}

func doCleanupForSnapshotCloneSrc(spdkClient *spdkclient.Client, c *SnapshotCloningSrcCache) (err error) {
	if c == nil || c.dstCloningBdevName == "" {
		return nil
	}
	if err := disconnectNVMfBdev(spdkClient, c.dstCloningBdevName, disconnectMaxRetries, disconnectRetryInterval); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return fmt.Errorf("failed to disconnect the cloning bdev %s for snapshot clone src cleanup", c.dstCloningBdevName)
	}
	return nil
}

func (r *Replica) SnapshotCloneSrcStatusCheck(spdkClient *spdkclient.Client, snapshotName, dstReplicaName string) (status *spdkrpc.ReplicaSnapshotCloneSrcStatusCheckResponse, err error) {
	r.Lock()
	defer r.Unlock()

	defer func() {
		err = errors.Wrapf(err, "failed to check snapshot clone status in src replica %v, dst replica %v, snapshot %v", r.Name, dstReplicaName, snapshotName)
	}()
	c := r.snapshotCloningSrcCache[dstReplicaName]
	if c == nil {
		return nil, fmt.Errorf("cannot find the operation in cache")
	}
	if c.snapshotName != snapshotName {
		return nil, fmt.Errorf("snapshot name %v is not the same as in the cache: %v", snapshotName, c.snapshotName)
	}
	if c.deepCopyOpID == 0 {
		return nil, fmt.Errorf("deepCopyOpID is empty in the cache")
	}

	// Only poll SPDK if not terminal.
	if c.deepCopyStatus.State != types.ProgressStateError && c.deepCopyStatus.State != types.ProgressStateComplete {
		s, err := spdkClient.BdevLvolCheckDeepCopy(c.deepCopyOpID)
		if err != nil {
			return nil, err
		}
		c.deepCopyStatus = DeepCopyStatus{
			State:             s.State,
			ProcessedClusters: s.ProcessedClusters,
			TotalClusters:     s.TotalClusters,
			Error:             s.Error,
		}
	}

	return &spdkrpc.ReplicaSnapshotCloneSrcStatusCheckResponse{
		State:             c.deepCopyStatus.State,
		ProcessedClusters: c.deepCopyStatus.ProcessedClusters,
		TotalClusters:     c.deepCopyStatus.TotalClusters,
		ErrorMsg:          c.deepCopyStatus.Error,
	}, nil
}

// SnapshotCloneSrcFinish asks the src replica to finish cloning (cleanup & drop cache)
func (r *Replica) SnapshotCloneSrcFinish(spdkClient *spdkclient.Client, dstReplicaName string) error {
	r.Lock()
	defer r.Unlock()

	c := r.snapshotCloningSrcCache[dstReplicaName]
	if c == nil {
		return nil
	}
	if err := doCleanupForSnapshotCloneSrc(spdkClient, c); err != nil {
		return err
	}
	delete(r.snapshotCloningSrcCache, dstReplicaName)
	return nil
}

// RebuildingSrcStart asks the source replica to check the parent snapshot of the head and expose it as a NVMf bdev if necessary.
// If the source replica and the destination replicas have different IPs, the API will expose the snapshot lvol as a NVMf bdev and return the address <IP>:<Port>.
// Otherwise, the API will directly return the snapshot lvol alias.
// It's not responsible for attaching rebuilding lvol of the dst replica.
func (r *Replica) RebuildingSrcStart(spdkClient *spdkclient.Client, dstReplicaName, dstReplicaAddress, exposedSnapshotName string) (exposedSnapshotLvolAddress string, err error) {
	updateRequired := false

	r.Lock()
	defer func() {
		r.Unlock()

		if updateRequired {
			r.UpdateCh <- nil
		}
	}()

	if r.State != types.InstanceStateRunning {
		return "", fmt.Errorf("invalid state %v for replica %s rebuilding src start", r.State, r.Name)
	}
	if r.isRebuilding {
		return "", fmt.Errorf("replica %s is being rebuilding hence it cannot be the source of rebuilding replica %s with snapshot %s", r.Name, dstReplicaName, exposedSnapshotName)
	}

	snapLvolName := GetReplicaSnapshotLvolName(r.Name, exposedSnapshotName)
	snapLvol := r.SnapshotLvolMap[snapLvolName]
	if snapLvol == nil {
		return "", fmt.Errorf("cannot find snapshot %s for the replica %s rebuilding src start", exposedSnapshotName, r.Name)
	}

	if r.rebuildingSrcCache.dstReplicaName != "" || r.rebuildingSrcCache.exposedSnapshotAlias != "" {
		if r.rebuildingSrcCache.dstReplicaName != dstReplicaName || r.rebuildingSrcCache.exposedSnapshotAlias != snapLvol.Alias {
			return "", fmt.Errorf("replica %s is helping rebuilding replica %s with the rebuilding snapshot %s, hence it cannot be the source of rebuilding replica %s with snapshot %s", r.Name, r.rebuildingSrcCache.dstReplicaName, r.rebuildingSrcCache.exposedSnapshotAlias, dstReplicaName, exposedSnapshotName)
		}
		if r.rebuildingSrcCache.exposedSnapshotPort != 0 {
			return net.JoinHostPort(r.IP, strconv.Itoa(int(r.rebuildingSrcCache.exposedSnapshotPort))), nil
		}
		// No exposed snapshot port, need to expose the snapshot lvol again
	}

	if err := r.stopSnapshotHash(spdkClient, snapLvol); err != nil {
		return "", errors.Wrapf(err, "failed to stop snapshot %s(%s) checksum hashing before replica %s rebuilding src exposes it", snapLvolName, exposedSnapshotName, r.Name)
	}

	port, _, err := r.portAllocator.AllocateRange(1)
	if err != nil {
		return "", err
	}
	if err := spdkClient.StartExposeBdev(helpertypes.GetNQN(snapLvol.Name), snapLvol.UUID, generateNGUID(snapLvol.Name), r.IP, strconv.Itoa(int(port))); err != nil {
		return "", err
	}
	exposedSnapshotLvolAddress = net.JoinHostPort(r.IP, strconv.Itoa(int(port)))

	r.rebuildingSrcCache.dstReplicaName = dstReplicaName
	r.rebuildingSrcCache.exposedSnapshotAlias = snapLvol.Alias
	r.rebuildingSrcCache.exposedSnapshotPort = port
	updateRequired = true

	r.log.Infof("Replica exposed snapshot %s(%s) to address %s for replica %s rebuilding start", exposedSnapshotName, snapLvol.UUID, exposedSnapshotLvolAddress, dstReplicaName)

	return exposedSnapshotLvolAddress, nil
}

// RebuildingSrcFinish asks the source replica to detach the rebuilding lvolof the dst replica, stop exposing the snapshot lvol (if necessary), and clean up the dst replica related cache
func (r *Replica) RebuildingSrcFinish(spdkClient *spdkclient.Client, dstReplicaName string) (err error) {
	updateRequired := false

	r.Lock()
	defer func() {
		r.Unlock()

		if updateRequired {
			r.UpdateCh <- nil
		}
	}()

	r.log.Infof("Replica is finishing rebuilding src for dst replica %s", dstReplicaName)

	if r.rebuildingSrcCache.dstReplicaName != "" && r.rebuildingSrcCache.dstReplicaName != dstReplicaName {
		return fmt.Errorf("found mismatching between the required dst replica name %s and the recorded dst replica name %s for replica %s rebuilding src finish", dstReplicaName, r.rebuildingSrcCache.dstReplicaName, r.Name)
	}

	r.doCleanupForRebuildingSrc(spdkClient)
	updateRequired = true

	return
}

func (r *Replica) doCleanupForRebuildingSrc(spdkClient *spdkclient.Client) {
	r.rebuildingSrcCache.shallowCopySnapshotName = ""
	r.rebuildingSrcCache.shallowCopyOpID = 0
	r.rebuildingSrcCache.shallowCopyStatus = ShallowCopyStatus{}

	if r.rebuildingSrcCache.dstRebuildingBdevName != "" {
		if err := disconnectNVMfBdev(spdkClient, r.rebuildingSrcCache.dstRebuildingBdevName, disconnectMaxRetries, disconnectRetryInterval); err != nil {
			r.log.WithError(err).Errorf("Failed to disconnect the rebuilding dst bdev %s for rebuilding src cleanup, will continue", r.rebuildingSrcCache.dstRebuildingBdevName)
		}
		// Always clear regardless of disconnect success or failure.
		// On same-node NVMe-oF, bdev_nvme_detach_controller may return ETIMEDOUT (-110).
		// Leaving the name set would cause ReplicaDelete to retry the detach and break the SPDK socket.
		r.rebuildingSrcCache.dstRebuildingBdevName = ""
	}

	if r.rebuildingSrcCache.exposedSnapshotPort != 0 {
		r.log.Infof("Replica is stopping exposing snapshot %s with port %d for rebuilding src cleanup", r.rebuildingSrcCache.exposedSnapshotAlias, r.rebuildingSrcCache.exposedSnapshotPort)
		if err := spdkClient.StopExposeBdev(helpertypes.GetNQN(spdktypes.GetLvolNameFromAlias(r.rebuildingSrcCache.exposedSnapshotAlias))); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			r.log.WithError(err).Errorf("Failed to stop exposing the snapshot %s for rebuilding src cleanup, will continue", r.rebuildingSrcCache.exposedSnapshotAlias)
		} else {
			if err := r.portAllocator.ReleaseRange(r.rebuildingSrcCache.exposedSnapshotPort, r.rebuildingSrcCache.exposedSnapshotPort); err != nil {
				r.log.WithError(err).Errorf("Failed to release exposed snapshot port %d for rebuilding src cleanup, will continue", r.rebuildingSrcCache.exposedSnapshotPort)
			} else {
				r.rebuildingSrcCache.exposedSnapshotPort = 0
				r.rebuildingSrcCache.exposedSnapshotAlias = ""
			}
		}
	}

	r.rebuildingSrcCache.dstReplicaName = ""
}

// rebuildingSrcAttachNoLock blindly attaches the rebuilding lvol of the dst replica as NVMf controller no matter if src and dst are on different nodes
func (r *Replica) rebuildingSrcAttachNoLock(spdkClient *spdkclient.Client, dstReplicaName, dstRebuildingLvolAddress string) (err error) {
	dstRebuildingLvolName := GetReplicaRebuildingLvolName(dstReplicaName)
	if r.rebuildingSrcCache.dstRebuildingBdevName != "" {
		controllerName := helperutil.GetNvmeControllerNameFromNamespaceName(r.rebuildingSrcCache.dstRebuildingBdevName)
		if dstRebuildingLvolName != controllerName {
			return fmt.Errorf("found mismatching between the required dst bdev NVMe controller name %s and the expected dst controller name %s for replica %s rebuilding src attach", dstRebuildingLvolName, controllerName, r.Name)
		}
		return nil
	}

	r.rebuildingSrcCache.dstRebuildingBdevName, err = connectNVMfBdev(spdkClient, dstRebuildingLvolName, dstRebuildingLvolAddress, replicaCtrlrLossTimeoutSec, replicaFastIOFailTimeoutSec, maxRetries, retryInterval)
	if err != nil {
		return errors.Wrapf(err, "failed to connect rebuilding lvol %s with address %s as a NVMe bdev for replica %s rebuilding src attach", dstRebuildingLvolName, dstRebuildingLvolAddress, r.Name)
	}

	return nil
}

// rebuildingSrcDetachNoLock detaches the rebuilding lvol of the dst replica as NVMf controller if src and dst are on different nodes
func (r *Replica) rebuildingSrcDetachNoLock(spdkClient *spdkclient.Client) (err error) {
	if r.rebuildingSrcCache.dstRebuildingBdevName == "" {
		return nil
	}
	if err := disconnectNVMfBdev(spdkClient, r.rebuildingSrcCache.dstRebuildingBdevName, disconnectMaxRetries, disconnectRetryInterval); err != nil {
		return err
	}
	r.rebuildingSrcCache.dstRebuildingBdevName = ""

	return nil
}

// RebuildingSrcShallowCopyStart asks the src replica to attach the dst rebuilding lvol, start a shallow copy from its snapshot lvol to it, then detach it.
func (r *Replica) RebuildingSrcShallowCopyStart(spdkClient *spdkclient.Client, snapshotName, dstRebuildingLvolAddress string) (err error) {
	r.Lock()
	defer r.Unlock()

	dstReplicaName := r.rebuildingSrcCache.dstReplicaName
	log := r.log.WithFields(logrus.Fields{"srcReplica": r.Name, "dstReplica": dstReplicaName, "snapshot": snapshotName, "dstRebuildingLvolAddress": dstRebuildingLvolAddress})

	if r.rebuildingSrcCache.shallowCopyStatus.State == types.ProgressStateInProgress || r.rebuildingSrcCache.shallowCopyStatus.State == types.ProgressStateStarting {
		return fmt.Errorf("cannot start a shallow copy from snapshot %s for the src replica %s since there is already a shallow copy starting or in progress", snapshotName, r.Name)
	}

	if err = r.rebuildingSrcDetachNoLock(spdkClient); err != nil {
		return errors.Wrapf(err, "failed to detach the rebuilding lvol of the dst replica %s before src replica %s shallow copy start", dstReplicaName, r.Name)
	}
	if err = r.rebuildingSrcAttachNoLock(spdkClient, dstReplicaName, dstRebuildingLvolAddress); err != nil {
		return errors.Wrapf(err, "failed to attach the rebuilding lvol of the dst replica %s before src replica %s shallow copy start", dstReplicaName, r.Name)
	}

	var shallowCopyOpID uint32
	defer func() {
		// TODO: May need to update r.rebuildingSrcCache.shallowCopyStatus
		if err != nil || shallowCopyOpID == 0 {
			return
		}
		go func() {
			timer := time.NewTimer(MaxShallowCopyWaitTime)
			defer timer.Stop()
			ticker := time.NewTicker(ShallowCopyCheckInterval)
			defer ticker.Stop()
			continuousRetryCount := 0
			for stopWaiting := false; !stopWaiting; {
				select {
				case <-timer.C:
					log.Errorf("Rebuilding src replica timeout waiting for shallow copy %v complete before detaching the dst replica rebuilding lvol, will give up", shallowCopyOpID)
					stopWaiting = true
					break // nolint: staticcheck
				case <-ticker.C:
					r.Lock()
					if r.rebuildingSrcCache.shallowCopyOpID != shallowCopyOpID || r.rebuildingSrcCache.shallowCopySnapshotName != snapshotName {
						r.Unlock()
						stopWaiting = true
						break
					}
					status, err := r.rebuildingSrcShallowCopyStatusUpdateAndHandlingNoLock(spdkClient)
					r.Unlock()
					if err != nil {
						continuousRetryCount++
						if continuousRetryCount > maxRetries {
							log.WithError(err).Errorf("Rebuilding src replica failed to check shallow copy %v status over %d times before detaching the dst replica rebuilding lvol, will give up", shallowCopyOpID, maxRetries)
							stopWaiting = true
							break
						}
						log.WithError(err).Errorf("Rebuilding src replica failed to check shallow copy %v status before detaching the dst replica rebuilding lvol, will retry later", shallowCopyOpID)
						continue
					}
					continuousRetryCount = 0
					if status.State == types.ProgressStateError || status.State == types.ProgressStateComplete {
						stopWaiting = true
						break // nolint: staticcheck
					}
				}
			}
		}()
	}()

	snapLvolName := GetReplicaSnapshotLvolName(r.Name, snapshotName)

	if r.rebuildingSrcCache.dstRebuildingBdevName == "" {
		return fmt.Errorf("no destination bdev for src replica %s shallow copy start", r.Name)
	}
	if r.SnapshotLvolMap[snapLvolName] == nil {
		return fmt.Errorf("cannot find snapshot %s for src replica %s shallow copy start", snapshotName, r.Name)
	}
	snapLvol := r.SnapshotLvolMap[snapLvolName]

	if err := r.stopSnapshotHash(spdkClient, snapLvol); err != nil {
		return errors.Wrapf(err, "failed to stop snapshot %s(%s) checksum hashing before replica %s rebuilding src starts shallow copy from it", snapLvolName, snapshotName, r.Name)
	}

	if shallowCopyOpID, err = spdkClient.BdevLvolStartShallowCopy(snapLvol.UUID, r.rebuildingSrcCache.dstRebuildingBdevName); err != nil {
		return err
	}
	r.rebuildingSrcCache.shallowCopySnapshotName = snapshotName
	r.rebuildingSrcCache.shallowCopyOpID = shallowCopyOpID
	r.rebuildingSrcCache.shallowCopyStatus = ShallowCopyStatus{}
	r.rebuildingSrcCache.isRangeShallowCopy = false

	if _, err = r.rebuildingSrcShallowCopyStatusUpdateAndHandlingNoLock(spdkClient); err != nil {
		return err
	}

	log.Infof("Rebuilding src replica started snapshot %s(%s)(%s) shallow copy %v to dst replica rebuilding bdev %s", snapshotName, snapLvol.Alias, snapLvol.UUID, shallowCopyOpID, r.rebuildingSrcCache.dstRebuildingBdevName)

	return
}

func (r *Replica) RebuildingSrcRangeShallowCopyStart(spdkClient *spdkclient.Client, snapshotName, dstRebuildingLvolAddress string, mismatchingClusterList []uint64) (err error) {
	var wg sync.WaitGroup

	r.Lock()
	defer func() {
		r.Unlock()

		// Wait for the first range shallow copy start before exit
		wg.Wait()
	}()
	log := r.log.WithFields(logrus.Fields{"srcReplica": r.Name, "dstReplica": r.rebuildingSrcCache.dstReplicaName, "snapshot": snapshotName, "dstRebuildingLvolAddress": dstRebuildingLvolAddress})

	if r.rebuildingSrcCache.shallowCopyStatus.State == types.ProgressStateInProgress || r.rebuildingSrcCache.shallowCopyStatus.State == types.ProgressStateStarting {
		return fmt.Errorf("cannot start a range shallow copy for rebuilding src replica %s snapshot %s since there is already a shallow copy starting or in progress", r.Name, snapshotName)
	}

	if err = r.rebuildingSrcDetachNoLock(spdkClient); err != nil {
		return errors.Wrapf(err, "failed to detach the rebuilding lvol of the dst replica %s before rebuilding src replica %s range shallow copy snapshot %s start", r.rebuildingSrcCache.dstReplicaName, r.Name, snapshotName)
	}
	if err = r.rebuildingSrcAttachNoLock(spdkClient, r.rebuildingSrcCache.dstReplicaName, dstRebuildingLvolAddress); err != nil {
		return errors.Wrapf(err, "failed to attach the rebuilding lvol of the dst replica %s before rebuilding src replica %s range shallow copy snapshot %s start", r.rebuildingSrcCache.dstReplicaName, r.Name, snapshotName)
	}

	snapLvolName := GetReplicaSnapshotLvolName(r.Name, snapshotName)

	if r.rebuildingSrcCache.dstRebuildingBdevName == "" {
		return fmt.Errorf("no destination bdev for rebuilding src replica %s range shallow copy snapshot %s start", r.Name, snapshotName)
	}
	if r.SnapshotLvolMap[snapLvolName] == nil {
		return fmt.Errorf("cannot find snapshot for rebuilding src replica %s range shallow copy snapshot %s start", r.Name, snapshotName)
	}
	snapSvcLvolAlias := r.SnapshotLvolMap[snapLvolName].Alias
	snapSvcLvolUUID := r.SnapshotLvolMap[snapLvolName].UUID

	r.rebuildingSrcCache.shallowCopySnapshotName = snapshotName
	r.rebuildingSrcCache.shallowCopyOpID = 0
	r.rebuildingSrcCache.shallowCopyStatus = ShallowCopyStatus{
		TotalClusters: uint64(len(mismatchingClusterList)),
	}
	r.rebuildingSrcCache.isRangeShallowCopy = true

	wg.Add(1)
	dstReplicaName := r.rebuildingSrcCache.dstReplicaName
	dstRebuildingBdevName := r.rebuildingSrcCache.dstRebuildingBdevName
	go func() {
		started := false
		var cpErr error
		defer func() {
			// TODO: May need to update r.rebuildingSrcCache.shallowCopyStatus
			if cpErr != nil {
				r.Lock()
				r.rebuildingDstCache.rebuildingError = cpErr.Error()
				r.rebuildingDstCache.rebuildingState = types.ProgressStateError
				r.rebuildingDstCache.processingState = types.ProgressStateError
				r.log.Error(cpErr)
				r.Unlock()
			} else {
				log.Debugf("Rebuilding src replica finished range shallow copy for snapshot %s(%s)(%s), mismatching clusters %+v", snapshotName, snapSvcLvolAlias, snapSvcLvolUUID, mismatchingClusterList)
			}
			if !started {
				wg.Done()
			}
		}()

		log.Debugf("Rebuilding src replica is starting range shallow copy for snapshot %s(%s)(%s), mismatching clusters %+v", snapshotName, snapSvcLvolAlias, snapSvcLvolUUID, mismatchingClusterList)
		head, tail, totalMismatchingClusterCount := uint64(0), lvolRangeShallowCopyLength, uint64(len(mismatchingClusterList))
		for head < totalMismatchingClusterCount {
			if tail > totalMismatchingClusterCount {
				tail = totalMismatchingClusterCount
			}

			r.Lock()
			if r.rebuildingSrcCache.shallowCopyStatus.State == types.ProgressStateError || r.rebuildingSrcCache.shallowCopyStatus.State == types.ProgressStateComplete {
				r.Unlock()
				cpErr = fmt.Errorf("failed to start a new range shallow copy in cluster range [%d, %d] to dst replica %s rebuilding bdev %s for rebuilding src replica %s range shallow copy snapshot %s(%s)(%s) since the state is unexpected changed to %v", mismatchingClusterList[head], mismatchingClusterList[tail-1], dstReplicaName, dstRebuildingBdevName, r.Name, snapshotName, snapSvcLvolAlias, snapSvcLvolUUID, r.rebuildingSrcCache.shallowCopyStatus.State)
				return
			}
			if r.rebuildingSrcCache.dstRebuildingBdevName != dstRebuildingBdevName {
				r.Unlock()
				cpErr = fmt.Errorf("failed to start a new range shallow copy in cluster range [%d, %d] to dst replica %s rebuilding bdev %s for rebuilding src replica %s range shallow copy snapshot %s(%s)(%s) since the recorded destination bdev name is changed from %s to %s", mismatchingClusterList[head], mismatchingClusterList[tail-1], dstReplicaName, dstRebuildingBdevName, r.Name, snapshotName, snapSvcLvolAlias, snapSvcLvolUUID, dstRebuildingBdevName, r.rebuildingSrcCache.dstRebuildingBdevName)
				return
			}
			if r.rebuildingSrcCache.shallowCopySnapshotName != snapshotName {
				r.Unlock()
				cpErr = fmt.Errorf("failed to start a new range shallow copy in cluster range [%d, %d] to dst replica %s rebuilding bdev %s for rebuilding src replica %s range shallow copy snapshot %s(%s)(%s) since the recorded rebuilding snapshot name is changed from %s to %s", mismatchingClusterList[head], mismatchingClusterList[tail-1], dstReplicaName, dstRebuildingBdevName, r.Name, snapshotName, snapSvcLvolAlias, snapSvcLvolUUID, snapshotName, r.rebuildingSrcCache.shallowCopySnapshotName)
				return
			}

			shallowCopyOpID, err := spdkClient.BdevLvolStartRangeShallowCopy(snapSvcLvolAlias, dstRebuildingBdevName, mismatchingClusterList[head:tail])
			if err != nil {
				r.Unlock()
				cpErr = errors.Wrapf(err, "failed to start range shallow copy in cluster range [%d, %d] to dst replica %s rebuilding bdev %s for rebuilding src replica %s range shallow copy snapshot %s(%s)(%s)", mismatchingClusterList[head], mismatchingClusterList[tail-1], dstReplicaName, dstRebuildingBdevName, r.Name, snapshotName, snapSvcLvolAlias, snapSvcLvolUUID)
				return
			}
			r.rebuildingSrcCache.shallowCopyOpID = shallowCopyOpID
			r.rebuildingSrcCache.shallowCopyStatus.CurrentRangeState = types.ProgressStateStarting
			r.Unlock()

			// Wait for the current range shallow copy to complete
			func() {
				timer := time.NewTimer(MaxShallowCopyWaitTime)
				defer timer.Stop()
				ticker := time.NewTicker(ShallowCopyCheckInterval)
				defer ticker.Stop()
				continuousRetryCount := 0
				for stopWaiting := false; !stopWaiting; {
					select {
					case <-timer.C:
						cpErr = fmt.Errorf("timeout waiting for rebuilding src replica %s range shallow copy snapshot %s(%s)(%s) with OP ID %d before detaching rebuilding dst replica %s rebuilding bdev %s, will give up", r.Name, snapshotName, snapSvcLvolAlias, snapSvcLvolUUID, shallowCopyOpID, dstReplicaName, dstRebuildingBdevName)
						stopWaiting = true
						break // nolint: staticcheck
					case <-ticker.C:
						r.Lock()
						if r.rebuildingSrcCache.shallowCopyStatus.State == types.ProgressStateError || r.rebuildingSrcCache.shallowCopyStatus.State == types.ProgressStateComplete {
							r.Unlock()
							stopWaiting = true
							break // nolint: staticcheck
						}

						status, err := r.rebuildingSrcShallowCopyStatusUpdateAndHandlingNoLock(spdkClient)
						r.Unlock()

						if err != nil {
							continuousRetryCount++
							if continuousRetryCount > maxRetries {
								cpErr = fmt.Errorf("failed to check the status over %d times for rebuilding src replica %s range shallow copy snapshot %s(%s)(%s) with OP ID %d before detaching rebuilding dst replica %s rebuilding bdev %s, will give up, last error: %v", maxRetries, r.Name, snapshotName, snapSvcLvolAlias, snapSvcLvolUUID, shallowCopyOpID, dstReplicaName, dstRebuildingBdevName, err)
								stopWaiting = true
								break
							}
							log.WithError(err).Warnf("Replica src replica failed to check the shallow copy status for snapshot %s(%s)(%s) with OP ID %d before detaching rebuilding dst replica %s rebuilding bdev %s, will retry later", snapshotName, snapSvcLvolAlias, snapSvcLvolUUID, shallowCopyOpID, dstReplicaName, dstRebuildingBdevName)
							continue
						}

						// Make sure the recorded shallow copy status is not empty before return
						if !started {
							started = true
							wg.Done()
						}

						continuousRetryCount = 0
						if status.CurrentRangeState == types.ProgressStateError || status.CurrentRangeState == types.ProgressStateComplete {
							if status.Error != "" {
								cpErr = fmt.Errorf("failed to do range shallow copy in cluster range [%d, %d] to dst replica %s rebuilding bdev %s for rebuilding src replica %s range shallow copy snapshot %s(%s)(%s) with OP ID %d, error: %s", mismatchingClusterList[head], mismatchingClusterList[tail-1], dstReplicaName, dstRebuildingBdevName, r.Name, snapshotName, snapSvcLvolAlias, snapSvcLvolUUID, shallowCopyOpID, status.Error)
							}
							stopWaiting = true
							break // nolint: staticcheck
						}
					}
				}
			}()
			if cpErr != nil {
				return
			}

			head = tail
			tail += lvolRangeShallowCopyLength
		}
	}()

	return
}

func (r *Replica) rebuildingSrcShallowCopyStatusUpdateAndHandlingNoLock(spdkClient *spdkclient.Client) (status ShallowCopyStatus, err error) {
	if r.rebuildingSrcCache.shallowCopyOpID == 0 {
		return ShallowCopyStatus{}, nil
	}
	// For a complete or errored shallow copy, spdk_tgt will clean up its status after the first check returns.
	// Hence we need to directly use the cached status here.
	// Similar logic applies to the range shallow copy, we need to check the cached status first.
	if r.rebuildingSrcCache.shallowCopyStatus.State == types.ProgressStateError || r.rebuildingSrcCache.shallowCopyStatus.State == types.ProgressStateComplete ||
		(r.rebuildingSrcCache.shallowCopyStatus.CurrentRangeState == types.ProgressStateComplete && r.rebuildingSrcCache.isRangeShallowCopy) {
		return r.rebuildingSrcCache.shallowCopyStatus, nil
	}

	currentShallowCopyStatus, err := spdkClient.BdevLvolCheckShallowCopy(r.rebuildingSrcCache.shallowCopyOpID)
	if err != nil {
		return ShallowCopyStatus{}, err
	}
	if currentShallowCopyStatus.State == types.SPDKShallowCopyStateInProgress {
		currentShallowCopyStatus.State = types.ProgressStateInProgress
	}

	r.rebuildingSrcCache.shallowCopyStatus.Error = currentShallowCopyStatus.Error
	if !r.rebuildingSrcCache.isRangeShallowCopy {
		r.rebuildingSrcCache.shallowCopyStatus.State = currentShallowCopyStatus.State
		r.rebuildingSrcCache.shallowCopyStatus.HandledClusters = currentShallowCopyStatus.CopiedClusters + currentShallowCopyStatus.UnmappedClusters
		r.rebuildingSrcCache.shallowCopyStatus.TotalClusters = currentShallowCopyStatus.TotalClusters
		// r.rebuildingSrcCache.shallowCopyStatus.CurrentRangeState and r.rebuildingSrcCache.shallowCopyStatus.HandledRangeClusters are not used for the full shallow copy
	} else {
		// For range shallow copy, the status returned from SPDK API involves the specific range of clusters only. We need to do calculation for the total progress.
		r.rebuildingSrcCache.shallowCopyStatus.HandledClusters = r.rebuildingSrcCache.shallowCopyStatus.HandledRangeClusters + currentShallowCopyStatus.CopiedClusters + currentShallowCopyStatus.UnmappedClusters
		r.rebuildingSrcCache.shallowCopyStatus.CurrentRangeState = currentShallowCopyStatus.State
		switch currentShallowCopyStatus.State {
		case types.ProgressStateError:
			r.rebuildingSrcCache.shallowCopyStatus.State = types.ProgressStateError
			r.rebuildingSrcCache.shallowCopyStatus.Error = currentShallowCopyStatus.Error
		case types.ProgressStateComplete:
			r.rebuildingSrcCache.shallowCopyStatus.HandledRangeClusters += currentShallowCopyStatus.TotalClusters
			if r.rebuildingSrcCache.shallowCopyStatus.HandledRangeClusters == r.rebuildingSrcCache.shallowCopyStatus.TotalClusters {
				r.rebuildingSrcCache.shallowCopyStatus.State = types.ProgressStateComplete
			} else {
				// The current range shallow copy completes while the whole snapshot copy is not done, the src replica should go to the next range
				r.rebuildingSrcCache.shallowCopyStatus.State = types.ProgressStateInProgress
			}
		case types.ProgressStateInProgress:
			r.rebuildingSrcCache.shallowCopyStatus.State = types.ProgressStateInProgress
		case types.ProgressStateStarting:
			if r.rebuildingSrcCache.shallowCopyStatus.HandledRangeClusters == 0 {
				r.rebuildingSrcCache.shallowCopyStatus.State = types.ProgressStateStarting
			}
		default:
			return ShallowCopyStatus{}, fmt.Errorf("found unknown shallow copy state %s for src replica %s shallow copy %v", currentShallowCopyStatus.State, r.Name, r.rebuildingSrcCache.shallowCopyOpID)
		}
	}

	// The status update and the detachment should be done atomically
	// Otherwise, the next shallow copy will be started before this detachment complete. In other words, the next shallow copy will be failed by this detachment.
	if r.rebuildingSrcCache.shallowCopyStatus.State == types.ProgressStateError || r.rebuildingSrcCache.shallowCopyStatus.State == types.ProgressStateComplete {
		err = r.rebuildingSrcDetachNoLock(spdkClient)
		if err != nil {
			r.log.WithError(err).Errorf("Rebuilding src replica failed to detach the rebuilding lvol of the dst replica %s after snapshot %s shallow copy %v finish, will continue", r.rebuildingSrcCache.dstReplicaName, r.rebuildingSrcCache.shallowCopySnapshotName, r.rebuildingSrcCache.shallowCopyOpID)
		}
	}

	return r.rebuildingSrcCache.shallowCopyStatus, nil
}

// RebuildingSrcShallowCopyCheck asks the src replica to check the shallow copy progress and status via the snapshot name.
func (r *Replica) RebuildingSrcShallowCopyCheck(snapshotName string) (status *spdkrpc.ReplicaRebuildingSrcShallowCopyCheckResponse, err error) {
	r.RLock()
	recordedSnapshotName := r.rebuildingSrcCache.shallowCopySnapshotName
	recordedShallowCopyStatus := r.rebuildingSrcCache.shallowCopyStatus
	r.RUnlock()

	if snapshotName != recordedSnapshotName {
		return nil, fmt.Errorf("found mismatching between the required snapshot name %v and the recorded snapshotName %v for src replica %s shallow copy check", snapshotName, recordedSnapshotName, r.Name)
	}

	return &spdkrpc.ReplicaRebuildingSrcShallowCopyCheckResponse{
		State:           recordedShallowCopyStatus.State,
		HandledClusters: recordedShallowCopyStatus.HandledClusters,
		TotalClusters:   recordedShallowCopyStatus.TotalClusters,
		ErrorMsg:        recordedShallowCopyStatus.Error,
	}, nil
}

// RebuildingDstStart asks the dst replica to create a new head lvol based on the external snapshot of the src replica and blindly expose it as a NVMf bdev.
// It returns the new head lvol address <IP>:<Port>.
// Notice that input `externalSnapshotAddress` is the alias of the external snapshot lvol if src and dst have on the same IP, otherwise it's the NVMf address of the external snapshot lvol.
func (r *Replica) RebuildingDstStart(spdkClient *spdkclient.Client, srcReplicaName, srcReplicaAddress, externalSnapshotName, externalSnapshotAddress string, rebuildingSnapshotList []*api.Lvol) (address string, err error) {
	updateRequired := false

	r.Lock()
	defer func() {
		r.Unlock()

		if updateRequired {
			r.UpdateCh <- nil
		}
	}()

	if r.State != types.InstanceStateRunning {
		return "", fmt.Errorf("invalid state %v for dst replica %s rebuilding start", r.State, r.Name)
	}
	if r.isRebuilding {
		return "", fmt.Errorf("replica %s rebuilding is in process", r.Name)
	}

	defer func() {
		if err != nil {
			if r.State != types.InstanceStateError {
				r.State = types.InstanceStateError
			}
			r.ErrorMsg = err.Error()
			if r.rebuildingDstCache.rebuildingError == "" {
				r.rebuildingDstCache.rebuildingError = err.Error()
				r.rebuildingDstCache.rebuildingState = types.ProgressStateError
			}
		} else {
			if r.State != types.InstanceStateError {
				r.ErrorMsg = ""
			}
		}

		updateRequired = true
	}()

	// Replica.Delete and Replica.Create do not guarantee that the previous rebuilding src replica info is cleaned up
	if r.rebuildingDstCache.srcReplicaName != "" || r.rebuildingDstCache.srcReplicaAddress != "" || r.rebuildingDstCache.externalSnapshotName != "" || r.rebuildingDstCache.externalSnapshotBdevName != "" {
		if err := r.doCleanupForRebuildingDst(spdkClient); err != nil {
			return "", errors.Wrapf(err, "failed to clean up the previous rebuilding dst info for dst replica rebuilding start, src replica name %s, address %s, external snapshot name %s, or external snapshot bdev name %s", r.rebuildingDstCache.srcReplicaName, r.rebuildingDstCache.srcReplicaAddress, r.rebuildingDstCache.externalSnapshotName, r.rebuildingDstCache.externalSnapshotBdevName)
		}
	}
	r.rebuildingDstCache.srcReplicaName = srcReplicaName
	r.rebuildingDstCache.srcReplicaAddress = srcReplicaAddress
	for _, apiLvol := range rebuildingSnapshotList {
		r.rebuildingDstCache.rebuildingSnapshotMap[apiLvol.Name] = apiLvol
		r.rebuildingDstCache.rebuildingSize += apiLvol.ActualSize
	}

	// Stop all in-progress snapshot hashing before connecting the external snapshot bdev.
	// Hash goroutines run without the replica lock and issue I/O on bdev channels.
	// If they are still running when the chain is restructured around the external snapshot,
	// the pending I/O can cause SPDK assertion failures during later cleanup.
	r.log.Info("Stopping all in-progress snapshot hashing before dst replica rebuilding start")
	if err := r.stopAllSnapshotHashing(spdkClient); err != nil {
		r.log.WithError(err).Warn("Failed to stop all snapshot hashing before rebuilding dst start, will continue")
	}

	externalSnapshotLvolName := GetReplicaSnapshotLvolName(srcReplicaName, externalSnapshotName)
	externalSnapshotBdevName, err := connectNVMfBdev(spdkClient, externalSnapshotLvolName, externalSnapshotAddress,
		replicaCtrlrLossTimeoutSec, replicaFastIOFailTimeoutSec, maxRetries, retryInterval)
	if err != nil {
		return "", errors.Wrapf(err, "failed to connect the external src snapshot lvol %s with address %s as a NVMf bdev for dst replica %v rebuilding start", externalSnapshotLvolName, externalSnapshotAddress, r.Name)
	}
	if r.rebuildingDstCache.externalSnapshotBdevName != "" && r.rebuildingDstCache.externalSnapshotBdevName != externalSnapshotBdevName {
		return "", fmt.Errorf("found mismatching between the required src snapshot bdev name %s and the expected src snapshot bdev name %s for dst replica %s rebuilding start", externalSnapshotBdevName, r.rebuildingDstCache.externalSnapshotBdevName, r.Name)
	}
	r.rebuildingDstCache.externalSnapshotName = externalSnapshotName
	r.rebuildingDstCache.externalSnapshotBdevName = externalSnapshotBdevName

	// Prepare a rebuilding port so that the dst replica can expose a rebuilding lvol in RebuildingDstSnapshotRevert
	if r.rebuildingDstCache.rebuildingPort == 0 {
		if r.rebuildingDstCache.rebuildingPort, _, err = r.portAllocator.AllocateRange(1); err != nil {
			return "", errors.Wrapf(err, "failed to allocate a rebuilding port for dst replica %v rebuilding start", r.Name)
		}
	}

	if r.IsExposed {
		r.log.Infof("Dst replica %s is already exposed before rebuilding start, will stop exposing it for rebuilding preparation", r.Name)
		if err := spdkClient.StopExposeBdev(helpertypes.GetNQN(r.Name)); err != nil {
			return "", err
		}
		r.IsExposed = false
	}
	// TODO: Uncomment below code after the RAID delta bitmap feature is ready
	//// For the old head, if it's a non-empty one, rename it for reuse later.
	//// Otherwise, directly remove it
	//if r.Head.ActualSize > 0 {
	//	expiredLvolName := GenerateReplicaExpiredLvolName(r.Name)
	//	if _, err := spdkClient.BdevLvolRename(r.Head.UUID, expiredLvolName); err != nil {
	//		r.log.WithError(err).Warnf("Failed to rename the previous head lvol %s to %s for dst replica %v rebuilding start, will try to remove it instead", r.Head.Alias, expiredLvolName, r.Name)
	//	}
	//}
	if _, err := spdkClient.BdevLvolDelete(r.Alias); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return "", err
	}

	// Retain the backing image in the active chain. All unverified lvols should be removed first.
	r.Head = nil
	r.ActiveChain = []*Lvol{r.ActiveChain[0]}

	// Create a new head lvol based on the external src snapshot lvol then
	headLvolUUID, err := spdkClient.BdevLvolCloneBdev(r.rebuildingDstCache.externalSnapshotBdevName, r.LvsName, r.Name)
	if err != nil {
		return "", err
	}
	headBdevLvol, err := spdkClient.BdevLvolGetByName(headLvolUUID, 0)
	if err != nil {
		return "", err
	}
	headBdevLvolSpecSize := headBdevLvol.NumBlocks * uint64(headBdevLvol.BlockSize)
	if headBdevLvolSpecSize != r.SpecSize {
		if _, err := spdkClient.BdevLvolResize(headBdevLvol.UUID, util.BytesToMiB(r.SpecSize)); err != nil {
			return "", errors.Wrapf(err, "failed to resize rebuilding dst head lvol %s from %d to %d", headBdevLvol.Name, headBdevLvolSpecSize, r.SpecSize)
		}
		headBdevLvol, err = spdkClient.BdevLvolGetByName(headLvolUUID, 0)
		if err != nil {
			return "", err
		}
	}
	r.Head = BdevLvolInfoToServiceLvol(&headBdevLvol)
	r.ActiveChain = append(r.ActiveChain, r.Head)

	if err := spdkClient.StartExposeBdev(helpertypes.GetNQN(r.Name), r.Head.UUID, generateNGUID(r.Name), r.IP, strconv.Itoa(int(r.PortStart))); err != nil {
		return "", err
	}
	r.IsExposed = true
	dstHeadLvolAddress := net.JoinHostPort(r.IP, strconv.Itoa(int(r.PortStart)))

	// Delete extra snapshots if any
	//rebuildingLvolName := GetReplicaRebuildingLvolName(r.Name)
	bdevLvolMap, err := GetBdevLvolMapWithFilter(spdkClient, r.replicaLvolFilter)
	if err != nil {
		return "", err
	}
	for lvolName, lvol := range bdevLvolMap {
		if types.IsBackingImageSnapLvolName(lvolName) {
			continue
		}
		if lvolName == r.Name {
			continue
		}
		if IsReplicaExpiredLvol(r.Name, lvolName) {
			continue
		}
		if r.rebuildingDstCache.rebuildingSnapshotMap[GetSnapshotNameFromReplicaSnapshotLvolName(r.Name, lvolName)] != nil {
			continue
		}

		// TODO: Uncomment below code after the RAID delta bitmap feature is ready
		//// Rename the non-empty previous rebuilding lvol so that it can be reused later
		//if lvolName == rebuildingLvolName {
		//	if lvol.DriverSpecific.Lvol.NumAllocatedClusters > 0 {
		//		expiredLvolName := GenerateReplicaExpiredLvolName(r.Name)
		//		if _, err := spdkClient.BdevLvolRename(lvol.UUID, expiredLvolName); err != nil {
		//			r.log.WithError(err).Warnf("Failed to rename the previous rebuilding lvol %s to %s for dst replica %v rebuilding start, will try to remove it instead", lvolName, expiredLvolName, r.Name)
		//		} else {
		//			continue
		//		}
		//	}
		//}

		// If an extra snapshot lvol has multiple children, decoupling it from its children before deletion
		if len(lvol.DriverSpecific.Lvol.Clones) > 1 {
			for _, childLvolName := range lvol.DriverSpecific.Lvol.Clones {
				if _, err := spdkClient.BdevLvolDetachParent(childLvolName); err != nil {
					return "", err
				}
			}
		}

		svcLvol := BdevLvolInfoToServiceLvol(lvol)
		if err := r.stopSnapshotHash(spdkClient, svcLvol); err != nil {
			return "", errors.Wrapf(err, "failed to stop redundant snapshot %s checksum hashing during rebuilding dst replica preparation", svcLvol.Name)
		}
		if _, err := spdkClient.BdevLvolDelete(svcLvol.UUID); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return "", err
		}
		r.log.Infof("Rebuilding dst replica found and deleted the redundant lvol %s(%s) during rebuilding dst replica preparation", svcLvol.Alias, svcLvol.UUID)
	}

	r.rebuildingDstCache.rebuildingError = ""
	r.rebuildingDstCache.rebuildingState = types.ProgressStateInProgress

	r.rebuildingDstCache.processingSnapshotName = ""
	r.rebuildingDstCache.processingState = types.ProgressStateStarting
	r.rebuildingDstCache.processingSize = 0
	r.rebuildingDstCache.processedSnapshotList = make([]string, 0, len(rebuildingSnapshotList))
	r.rebuildingDstCache.processedSnapshotsSize = 0

	r.isRebuilding = true

	r.log.Infof("Rebuilding dst replica created a new head %s(%s) based on the external snapshot %s(%s)(%s) from src replica %s for rebuilding start", r.Head.Alias, dstHeadLvolAddress, externalSnapshotName, r.rebuildingDstCache.externalSnapshotBdevName, externalSnapshotAddress, srcReplicaName)

	return dstHeadLvolAddress, nil
}

// RebuildingDstFinish asks the dst replica to switch the parent of earliest lvol of the dst replica from the external src snapshot to the rebuilt snapshot then detach that external src snapshot (if necessary).
// The engine should guarantee that there is no IO during the parent switch.
func (r *Replica) RebuildingDstFinish(spdkClient *spdkclient.Client) (err error) {
	updateRequired := false

	r.Lock()
	defer func() {
		r.Unlock()

		if updateRequired {
			r.UpdateCh <- nil
		}
	}()

	if r.State != types.InstanceStateRunning && r.State != types.InstanceStateError {
		return fmt.Errorf("invalid state %v for replica %s rebuilding finish", r.State, r.Name)
	}
	if !r.isRebuilding {
		return fmt.Errorf("replica %s is not in rebuilding", r.Name)
	}

	defer func() {
		if err != nil {
			if r.State != types.InstanceStateError {
				r.State = types.InstanceStateError
			}
			r.ErrorMsg = err.Error()
			if r.rebuildingDstCache.rebuildingError == "" {
				r.rebuildingDstCache.rebuildingError = err.Error()
				r.rebuildingDstCache.rebuildingState = types.ProgressStateError
			}
		} else {
			if r.State != types.InstanceStateError {
				r.ErrorMsg = ""
			}
		}

		// Mark the rebuilding as complete after construction done
		r.isRebuilding = false

		updateRequired = true
	}()

	if len(r.ActiveChain) < 2 {
		return fmt.Errorf("invalid chain length %d for dst replica %v rebuilding finish", len(r.ActiveChain), r.Name)
	}

	// Switch from the external snapshot to use rebuilt snapshots
	var setParentErr error
	if r.rebuildingDstCache.rebuildingError == "" {
		// Probably this lvol is the head
		firstLvolAfterRebuilding := r.ActiveChain[1]
		if firstLvolAfterRebuilding == nil {
			return fmt.Errorf("cannot find the head or the first snapshot since rebuilding start for replica %s rebuilding finish", r.Name)
		}
		if _, setParentErr = spdkClient.BdevLvolSetParent(firstLvolAfterRebuilding.Alias, spdktypes.GetLvolAlias(r.LvsName, GetReplicaSnapshotLvolName(r.Name, r.rebuildingDstCache.externalSnapshotName))); setParentErr != nil {
			r.log.WithError(setParentErr).Errorf("Rebuilding dst replica %s failed to set parent, will continue with cleanup", r.Name)
		} else {
			firstLvolAfterRebuilding.Parent = GetReplicaSnapshotLvolName(r.Name, r.rebuildingDstCache.externalSnapshotName)
		}

		if r.rebuildingDstCache.processedSnapshotsSize != r.rebuildingDstCache.rebuildingSize {
			r.log.Warnf("Rebuilding dst replica detected that the rebuilding spec size %d does not match the total processed snapshots size %d when during the dst rebuilding finish", r.rebuildingDstCache.rebuildingSize, r.rebuildingDstCache.processedSnapshotsSize)
			r.rebuildingDstCache.processedSnapshotsSize = r.rebuildingDstCache.rebuildingSize
		}
	}

	// Always perform cleanup to disconnect the external snapshot NVMe controller.
	// Without this, subsequent ReplicaDelete would trigger bdev_nvme_detach_controller
	// which can hang on same-node NVMe-oF connections.
	_ = r.doCleanupForRebuildingDst(spdkClient)

	// If setParent failed, propagate the error after cleanup is done
	if setParentErr != nil {
		return setParentErr
	}

	bdevLvolMap, err := GetBdevLvolMapWithFilter(spdkClient, r.replicaLvolFilter)
	if err != nil {
		return err
	}
	if err = r.construct(bdevLvolMap); err != nil {
		return err
	}

	r.rebuildingDstCache.processingState = types.ProgressStateComplete
	r.rebuildingDstCache.rebuildingState = types.ProgressStateComplete
	r.lastRebuildingAt = time.Now()

	return nil
}

// doCleanupForRebuildingDst blindly cleans up the dst replica rebuilding cache and all redundant lvols if any
// Option cleanupRequired should be set to true if Longhorn does not want to reuse this dst replica for the next fast rebuilding, which typically means the replica removal
func (r *Replica) doCleanupForRebuildingDst(spdkClient *spdkclient.Client) error {
	aggregatedErrors := []error{}

	// Ensure all snapshot hash operations on rebuilding snapshots and existing snapshots are stopped
	// before disconnecting the external snapshot bdev, to avoid assertion failure caused by pending I/O.
	_ = r.stopAllSnapshotHashing(spdkClient)

	if r.rebuildingDstCache.externalSnapshotBdevName != "" {
		if err := disconnectNVMfBdev(spdkClient, r.rebuildingDstCache.externalSnapshotBdevName, disconnectMaxRetries, disconnectRetryInterval); err != nil {
			r.log.WithError(err).Errorf("Rebuilding dst replica failed to disconnect the external src snapshot bdev %s for rebuilding dst cleanup, will continue", r.rebuildingDstCache.externalSnapshotBdevName)
			aggregatedErrors = append(aggregatedErrors, err)
		}
		// Always clear the cache fields regardless of disconnect success or failure.
		// On same-node NVMe-oF, bdev_nvme_detach_controller returns ETIMEDOUT (-110)
		// which is survivable (SPDK socket remains functional). But if we leave
		// externalSnapshotBdevName set, subsequent calls (e.g. from ReplicaDelete)
		// will retry the detach, causing a second ETIMEDOUT that breaks the SPDK socket.
		r.rebuildingDstCache.srcReplicaName = ""
		r.rebuildingDstCache.srcReplicaAddress = ""
		r.rebuildingDstCache.externalSnapshotName = ""
		r.rebuildingDstCache.externalSnapshotBdevName = ""
	}

	// Blindly clean up the rebuilding lvol and the exposed port
	rebuildingLvolName := GetReplicaRebuildingLvolName(r.Name)
	if r.rebuildingDstCache.rebuildingLvol != nil && r.rebuildingDstCache.rebuildingLvol.Name != rebuildingLvolName {
		err := fmt.Errorf("BUG: replica %s rebuilding lvol actual name %s does not match the expected name %v, will use the actual name for the cleanup", r.Name, r.rebuildingDstCache.rebuildingLvol.Name, rebuildingLvolName)
		r.log.Error(err)
		aggregatedErrors = append(aggregatedErrors, err)
		rebuildingLvolName = r.rebuildingDstCache.rebuildingLvol.Name
	}
	r.log.Infof("Rebuilding dst replica starts to blindly clean up the rebuilding lvol %s and the exposed port %d for rebuilding dst cleanup", rebuildingLvolName, r.rebuildingDstCache.rebuildingPort)
	if err := spdkClient.StopExposeBdev(helpertypes.GetNQN(rebuildingLvolName)); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		r.log.WithError(err).Errorf("Rebuilding dst replica failed to stop exposing the rebuilding lvol %s for rebuilding dst cleanup, will continue", rebuildingLvolName)
		aggregatedErrors = append(aggregatedErrors, err)
	}
	if r.rebuildingDstCache.rebuildingPort != 0 {
		if err := r.portAllocator.ReleaseRange(r.rebuildingDstCache.rebuildingPort, r.rebuildingDstCache.rebuildingPort); err != nil {
			r.log.WithError(err).Errorf("Rebuilding dst replica failed to release the rebuilding port %d for rebuilding dst cleanup, will continue", r.rebuildingDstCache.rebuildingPort)
			aggregatedErrors = append(aggregatedErrors, err)
		} else {
			r.rebuildingDstCache.rebuildingPort = 0
			r.rebuildingDstCache.rebuildingLvolAddress = ""
		}
	}
	if _, err := spdkClient.BdevLvolDelete(spdktypes.GetLvolAlias(r.LvsName, rebuildingLvolName)); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		r.log.WithError(err).Errorf("Rebuilding dst replica failed to delete the rebuilding lvol %s for rebuilding dst cleanup, will continue", rebuildingLvolName)
		aggregatedErrors = append(aggregatedErrors, err)
	} else {
		r.rebuildingDstCache.rebuildingLvol = nil
	}

	// Remove redundant lvols if any at the end of a rebuilding.
	if len(r.rebuildingDstCache.rebuildingSnapshotMap) > 0 {
		bdevLvolMap, err := GetBdevLvolMapWithFilter(spdkClient, r.replicaLvolFilter)
		if err != nil {
			return err
		}
		chainLvolMap := map[string]*Lvol{}
		for _, inChainLvol := range r.ActiveChain {
			if inChainLvol == nil {
				continue
			}
			chainLvolMap[inChainLvol.Name] = inChainLvol
		}
		for lvolName, lvol := range bdevLvolMap {
			if types.IsBackingImageSnapLvolName(lvolName) {
				continue
			}
			if lvolName == r.Name || IsRebuildingLvol(lvolName) || IsReplicaExpiredLvol(r.Name, lvolName) {
				continue
			}
			if chainLvolMap[lvolName] != nil {
				continue
			}
			if r.rebuildingDstCache.rebuildingSnapshotMap[GetSnapshotNameFromReplicaSnapshotLvolName(r.Name, lvolName)] != nil {
				continue
			}
			if len(lvol.DriverSpecific.Lvol.Clones) > 1 {
				for _, childLvolName := range lvol.DriverSpecific.Lvol.Clones {
					if childLvolName == r.Name || r.rebuildingDstCache.rebuildingSnapshotMap[GetSnapshotNameFromReplicaSnapshotLvolName(r.Name, childLvolName)] != nil {
						return fmt.Errorf("found a valid lvol %s in the redundant lvol %s children list for replica %s rebuilding cleanup", childLvolName, lvolName, r.Name)
					}
					if _, err := spdkClient.BdevLvolDetachParent(childLvolName); err != nil {
						return err
					}
				}
			}
			if _, err := spdkClient.BdevLvolDelete(lvol.Aliases[0]); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
				return err
			}
			r.log.Infof("Rebuilding dst replica found and deleted the redundant lvol %s(%s) for dst replica %v rebuilding cleanup", lvol.Aliases[0], lvol.UUID, r.Name)
		}
	}

	r.rebuildingDstCache.rebuildingSnapshotMap = map[string]*api.Lvol{}
	r.rebuildingDstCache.rebuildingSize = 0
	r.rebuildingDstCache.rebuildingError = ""
	r.rebuildingDstCache.rebuildingState = ""
	r.rebuildingDstCache.processedSnapshotList = make([]string, 0)
	r.rebuildingDstCache.processedSnapshotsSize = 0
	r.rebuildingDstCache.processingSnapshotName = ""
	r.rebuildingDstCache.processingSize = 0
	r.rebuildingDstCache.processingState = ""

	return util.CombineErrors(aggregatedErrors...)
}

// rebuildingDstShallowCopyPrepare creates a new rebuilding lvol or renames an existing expired lvol as the rebuilding lvol for the dst replica.
func (r *Replica) rebuildingDstShallowCopyPrepare(spdkClient *spdkclient.Client, srcReplicaServiceCli *client.SPDKClient, snapshotName string, fastSync bool) (dstRebuildingLvolAddress string, requireRangeCopy bool, err error) {
	r.log.Infof("Rebuilding dst replica %s starts to prepare the rebuilding lvol for snapshot %s shallow copy", r.Name, snapshotName)

	rebuildingLvolName := GetReplicaRebuildingLvolName(r.Name)

	dstSnapshotParentLvolName := ""
	srcSnapSvcLvol := r.rebuildingDstCache.rebuildingSnapshotMap[snapshotName]
	if srcSnapSvcLvol == nil {
		return "", false, fmt.Errorf("cannot find snapshot %s in the rebuilding snapshot list for replica %s shallow copy prepare", snapshotName, r.Name)
	}

	// For the ancestor snapshot of the rebuilding snapshot list, its parent will not record the backing image info
	if srcSnapSvcLvol.Parent == "" {
		if r.BackingImage != nil {
			dstSnapshotParentLvolName = r.BackingImage.Name
		}
	} else {
		dstSnapshotParentLvolName = GetReplicaSnapshotLvolName(r.Name, srcSnapSvcLvol.Parent)
	}

	// Blindly clean up the existing rebuilding lvol
	if r.rebuildingDstCache.rebuildingPort != 0 {
		r.log.Infof("Rebuilding dst replica %s stops exposing the rebuilding lvol on port %d for snapshot %s shallow copy prepare", r.Name, r.rebuildingDstCache.rebuildingPort, snapshotName)
		if err := spdkClient.StopExposeBdev(helpertypes.GetNQN(rebuildingLvolName)); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return "", false, err
		}
	}
	if r.rebuildingDstCache.rebuildingLvol != nil {
		r.log.Infof("Rebuilding dst replica %s deletes the existing rebuilding lvol %s(%s) for snapshot %s shallow copy prepare", r.Name, r.rebuildingDstCache.rebuildingLvol.Name, r.rebuildingDstCache.rebuildingLvol.UUID, snapshotName)
		if _, err := spdkClient.BdevLvolDelete(r.rebuildingDstCache.rebuildingLvol.UUID); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return "", false, err
		}
	}
	r.rebuildingDstCache.rebuildingLvol = nil
	rebuildingLvolCreated := false

	bdevLvolMap, err := GetBdevLvolMapWithFilter(spdkClient, r.replicaLvolFilter)
	if err != nil {
		return "", false, err
	}
	dstSnapshotLvolName := GetReplicaSnapshotLvolName(r.Name, snapshotName)
	if bdevLvolMap[dstSnapshotLvolName] != nil { // If there is an existing snapshot lvol, clone a rebuilding lvol behinds it
		// Otherwise, try to reuse the existing lvol
		dstSnapSvcLvol := BdevLvolInfoToServiceLvol(bdevLvolMap[dstSnapshotLvolName])
		if err := r.stopSnapshotHash(spdkClient, dstSnapSvcLvol); err != nil {
			return "", false, errors.Wrapf(err, "failed to stop the existing snapshot %s checksum hashing before rebuilding dst replica reuses then clones a rebuilding lvol behind it", dstSnapshotLvolName)
		}
		isIntactSnap := srcSnapSvcLvol.SnapshotTimestamp == dstSnapSvcLvol.SnapshotTimestamp &&
			srcSnapSvcLvol.ActualSize == dstSnapSvcLvol.ActualSize &&
			srcSnapSvcLvol.SnapshotChecksum != "" && dstSnapSvcLvol.SnapshotChecksum != "" && srcSnapSvcLvol.SnapshotChecksum == dstSnapSvcLvol.SnapshotChecksum

		if isIntactSnap && fastSync {
			if _, err = spdkClient.BdevLvolClone(dstSnapSvcLvol.Alias, rebuildingLvolName); err != nil {
				return "", false, errors.Wrapf(err, "failed to clone rebuilding lvol %s behinds the existing intact snapshot lvol %s for dst replica %v rebuilding snapshot %s shallow copy prepare", rebuildingLvolName, dstSnapSvcLvol.Alias, r.Name, snapshotName)
			}
			rebuildingLvolCreated = true
			r.log.Infof("Rebuilding dst replica found an intact snapshot lvol %s(%s) before the shallow copy", dstSnapSvcLvol.Alias, dstSnapSvcLvol.UUID)
		} else {
			// For an existing but corrupted or outdated snapshot lvol:
			// 1. If it contains the range checksums, SPDK server will reuse it later.
			// 2. If not, SPDK server should delete it then do full rebuilding to a brand new rebuilding lvol.
			for childLvolName := range dstSnapSvcLvol.Children {
				if _, err := spdkClient.BdevLvolDetachParent(spdktypes.GetLvolAlias(r.LvsName, childLvolName)); err != nil {
					return "", false, errors.Wrapf(err, "failed to decouple the child lvol %s from the corrupted or outdated snapshot lvol %s for dst replica %v rebuilding snapshot %s shallow copy prepare", childLvolName, dstSnapshotLvolName, r.Name, snapshotName)
				}
			}
			if r.isEligibleForRebuildingDstForRangeShallowCopy(spdkClient, srcReplicaServiceCli, snapshotName) {
				// For reusable corrupted or outdated snapshot lvol, we will clone the rebuilding lvol first. The snapshot deletion will be delayed until the range shallow copy is complete.
				// The reason for detaching the parent of reusable corrupted or outdated snapshot lvol is, spdk_tgt requires the rebuilding lvol being backed by a zeros bdev so that it can release clusters after unmapping the rebuilding lvol.
				if dstSnapSvcLvol.Parent != "" {
					if _, err := spdkClient.BdevLvolDetachParent(dstSnapSvcLvol.Alias); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
						return "", false, errors.Wrapf(err, "failed to detach the corrupted or outdated snapshot lvol %s from its parent for dst replica %v rebuilding snapshot %s shallow copy prepare", dstSnapshotLvolName, r.Name, snapshotName)
					}
				}
				if _, err = spdkClient.BdevLvolClone(dstSnapSvcLvol.Alias, rebuildingLvolName); err != nil {
					return "", false, errors.Wrapf(err, "failed to clone rebuilding lvol %s behinds the existing snapshot lvol %s for dst replica %v rebuilding snapshot %s shallow copy prepare", rebuildingLvolName, dstSnapSvcLvol.Alias, r.Name, snapshotName)
				}
				rebuildingLvolCreated = true
				requireRangeCopy = true
				r.log.Infof("Rebuilding dst replica found the reusable corrupted or outdated snapshot lvol %s(%s) hence cloned the rebuilding lvol %s before the shallow copy", dstSnapSvcLvol.Alias, dstSnapSvcLvol.UUID, rebuildingLvolName)
			} else {
				r.log.Infof("Rebuilding dst replica found the non-reusable corrupted or outdated snapshot lvol %s(%s) before the shallow copy", dstSnapSvcLvol.Alias, dstSnapSvcLvol.UUID)
				if _, err = spdkClient.BdevLvolDelete(dstSnapSvcLvol.Alias); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
					return "", false, errors.Wrapf(err, "failed to delete the non-reusable corrupted or outdated snapshot lvol %s for dst replica %v rebuilding snapshot %s shallow copy prepare", dstSnapshotLvolName, r.Name, snapshotName)
				}
				r.log.Infof("Rebuilding dst replica found the non-reusable corrupted or outdated snapshot lvol %s(%s) and deleted it before the shallow copy", dstSnapSvcLvol.Alias, dstSnapSvcLvol.UUID)
			}
		}
		// TODO: Uncomment and modify the below code when SPDK server could verify and reuse the non-snapshot orphan lvol as rebuildng lvol to speed up the progress. (RAID delta bitmap feature)
		//} else if bdevLvolMap[dstSnapshotParentLvolName] != nil {
		//	// For ancestor snapshot, it's hard to quickly figure out if there is an orphan lvol that probably contains the data. Hence we will give up range shallow copy for this case.
		//	// For non-ancestor snapshot, we can check if dstSnapshotParentLvol has only one expired lvol as child. If YES, this is probably the corrupted snapshot.
		//	onlyExpiredChildLvolName := ""
		//	for _, childLvolName := range bdevLvolMap[dstSnapshotParentLvolName].DriverSpecific.Lvol.Clones {
		//		if IsReplicaExpiredLvol(r.Name, childLvolName) {
		//			if onlyExpiredChildLvolName == "" {
		//				onlyExpiredChildLvolName = childLvolName
		//			} else {
		//				onlyExpiredChildLvolName = ""
		//				break
		//			}
		//		}
		//	}
		//	if onlyExpiredChildLvolName != "" {
		//		if r.isEligibleForRebuildingDstForRangeShallowCopy(spdkClient, srcReplicaServiceCli, onlyExpiredChildLvolName, snapshotName) {
		//			if _, err := spdkClient.BdevLvolRename(spdktypes.GetLvolAlias(r.LvsName, onlyExpiredChildLvolName), rebuildingLvolName); err != nil {
		//				r.log.WithError(err).Warnf("Failed to rename the previous expired lvol %s to rebuilding lvol %s for dst replica %v rebuilding snapshot %s shallow copy prepare, will ignore it and continue", onlyExpiredChildLvolName, rebuildingLvolName, r.Name, snapshotName)
		//			} else {
		//				rebuildingLvolCreated = true
		//				requireRangeCopy = true
		//				r.log.Infof("Replica found an expired lvol %s (with parent %s) and renamed it to rebuilding lvol %s for dst replica %v rebuilding snapshot %s shallow copy prepare", onlyExpiredChildLvolName, dstSnapshotParentLvolName, rebuildingLvolName, r.Name, snapshotName)
		//			}
		//		}
		//	}
	}

	if !rebuildingLvolCreated {
		if dstSnapshotParentLvolName != "" && bdevLvolMap[dstSnapshotParentLvolName] != nil {
			dstSnapshotParentSvcLvol := BdevLvolInfoToServiceLvol(bdevLvolMap[dstSnapshotParentLvolName])
			if err := r.stopSnapshotHash(spdkClient, dstSnapshotParentSvcLvol); err != nil {
				return "", false, errors.Wrapf(err, "failed to stop snapshot %s checksum hashing before rebuilding dst replica prepares a rebuilding lvol behind it", dstSnapshotParentLvolName)
			}
			if _, err = spdkClient.BdevLvolClone(dstSnapshotParentSvcLvol.Alias, rebuildingLvolName); err != nil {
				return "", false, err
			}
		} else {
			if _, err = spdkClient.BdevLvolCreate("", r.LvsUUID, rebuildingLvolName, util.BytesToMiB(r.SpecSize), "", true); err != nil {
				return "", false, err
			}
		}
	}

	rebuildingLvolAlias := spdktypes.GetLvolAlias(r.LvsName, rebuildingLvolName)
	rebuildingBdevLvol, err := spdkClient.BdevLvolGetByName(rebuildingLvolAlias, 0)
	if err != nil {
		return "", false, err
	}
	r.rebuildingDstCache.rebuildingLvol = BdevLvolInfoToServiceLvol(&rebuildingBdevLvol)

	targetRebuildingLvolSize := srcSnapSvcLvol.SpecSize
	if snapshotName == r.rebuildingDstCache.externalSnapshotName && targetRebuildingLvolSize < r.SpecSize {
		targetRebuildingLvolSize = r.SpecSize
	}
	if targetRebuildingLvolSize != r.rebuildingDstCache.rebuildingLvol.SpecSize {
		if _, err := spdkClient.BdevLvolResize(r.rebuildingDstCache.rebuildingLvol.Alias, util.BytesToMiB(targetRebuildingLvolSize)); err != nil {
			return "", false, err
		}
		r.rebuildingDstCache.rebuildingLvol.SpecSize = targetRebuildingLvolSize
	}

	// Apply QoS limit if set
	if r.rebuildingQosLimitMbps > 0 {
		if err := spdkClient.BdevSetQosLimit(r.rebuildingDstCache.rebuildingLvol.UUID, 0, 0, 0, r.rebuildingQosLimitMbps); err != nil {
			return "", false, err
		}
		r.log.Infof("Rebuilding dst replica applied QoS limit %d MB/s to new rebuilding lvol %s", r.rebuildingQosLimitMbps, r.rebuildingDstCache.rebuildingLvol.UUID)
	}

	dstRebuildingLvolAddress = r.rebuildingDstCache.rebuildingLvol.Alias
	if r.rebuildingDstCache.rebuildingPort != 0 {
		if err := spdkClient.StartExposeBdev(helpertypes.GetNQN(r.rebuildingDstCache.rebuildingLvol.Name), r.rebuildingDstCache.rebuildingLvol.UUID,
			generateNGUID(r.rebuildingDstCache.rebuildingLvol.Name), r.IP, strconv.Itoa(int(r.rebuildingDstCache.rebuildingPort))); err != nil {
			return "", false, err
		}
		dstRebuildingLvolAddress = net.JoinHostPort(r.IP, strconv.Itoa(int(r.rebuildingDstCache.rebuildingPort)))
	}
	r.rebuildingDstCache.rebuildingLvolAddress = dstRebuildingLvolAddress

	r.log.Infof("Rebuilding dst replica prepared its rebuilding lvol %s(%s) with parent %s for snapshot %s and expose it to %s", r.rebuildingDstCache.rebuildingLvol.Alias, r.rebuildingDstCache.rebuildingLvol.UUID, r.rebuildingDstCache.rebuildingLvol.Parent, snapshotName, dstRebuildingLvolAddress)

	return dstRebuildingLvolAddress, requireRangeCopy, nil
}

func (r *Replica) isEligibleForRebuildingDstForRangeShallowCopy(spdkClient *spdkclient.Client, srcReplicaServiceCli *client.SPDKClient, snapshotName string) bool {
	if _, err := spdkClient.BdevLvolGetRangeChecksums(spdktypes.GetLvolAlias(r.LvsName, GetReplicaSnapshotLvolName(r.Name, snapshotName)), 0, 1); err != nil {
		return false
	}
	if _, err := srcReplicaServiceCli.ReplicaSnapshotRangeHashGet(r.rebuildingDstCache.srcReplicaName, snapshotName, 0, 1); err != nil {
		return false
	}
	return true
}

func (r *Replica) rebuildingDstRangeShallowCopy(spdkClient *spdkclient.Client, srcReplicaServiceCli *client.SPDKClient, snapshotName, dstRebuildingLvolAddress string) (err error) {
	totalClusterCount := uint64(r.SpecSize / defaultClusterSize)
	offset, count := uint64(0), lvolRangeShallowCopyLength
	dstSnapshotLvolName := GetReplicaSnapshotLvolName(r.Name, snapshotName)
	dstSnapshotLvolAlias := spdktypes.GetLvolAlias(r.LvsName, dstSnapshotLvolName)
	rebuildingLvolName := GetReplicaRebuildingLvolName(r.Name)

	r.log.Debugf("Rebuilding dst replica is starting snapshot %s range shallow copy from src replica %s to dst replica rebuilding lvol with address %s", snapshotName, r.rebuildingDstCache.srcReplicaName, dstRebuildingLvolAddress)

	mismatchingClusterList := make([]uint64, 0, count)
	for offset < totalClusterCount {
		if offset+count > totalClusterCount {
			count = totalClusterCount - offset
		}
		dstReplicaRangeHashMap, err := spdkClient.BdevLvolGetRangeChecksums(dstSnapshotLvolAlias, offset, count)
		if err != nil {
			return errors.Wrapf(err, "failed to get the range [%d, %d) cluster checksums from dst replica %s for rebuilding dst replica %v snapshot %s range shallow copy", offset, offset+count, r.Name, r.Name, snapshotName)
		}
		srcReplicaRangeHashResp, err := srcReplicaServiceCli.ReplicaSnapshotRangeHashGet(r.rebuildingDstCache.srcReplicaName, snapshotName, offset, count)
		if err != nil {
			return errors.Wrapf(err, "failed to get the range [%d, %d) snapshot checksums from src replica %s for rebuilding dst replica %v snapshot %s range shallow copy", offset, offset+count, r.rebuildingDstCache.srcReplicaName, r.Name, snapshotName)
		}
		for idx := range srcReplicaRangeHashResp.RangeHashMap {
			if dstReplicaRangeHashMap[idx] != srcReplicaRangeHashResp.RangeHashMap[idx] {
				mismatchingClusterList = append(mismatchingClusterList, idx)
			}
		}
		offset += count
	}
	slices.Sort(mismatchingClusterList)

	// Need to delete the corrupted or outdated snapshot lvol so that its data will be merged into the rebuilding lvol
	// Then Later on the range shallow copy will correct or unmap the corrupted or outdated data for the rebuilding lvol
	if _, err := spdkClient.BdevLvolDelete(dstSnapshotLvolAlias); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return errors.Wrapf(err, "failed to delete the corrupted or outdated snapshot lvol %s for rebuilding dst replica %v snapshot %s range shallow copy", dstSnapshotLvolAlias, r.Name, snapshotName)
	}

	rebuildingBdevLvol, err := spdkClient.BdevLvolGetByName(spdktypes.GetLvolAlias(r.LvsName, rebuildingLvolName), 0)
	if err != nil {
		return errors.Wrapf(err, "failed to get the rebuilding lvol %s for rebuilding dst replica %v snapshot %s range shallow copy", r.rebuildingDstCache.rebuildingLvol.Alias, r.Name, snapshotName)
	} else {
		r.rebuildingDstCache.rebuildingLvol = BdevLvolInfoToServiceLvol(&rebuildingBdevLvol)
	}

	return srcReplicaServiceCli.ReplicaRebuildingSrcRangeShallowCopyStart(r.rebuildingDstCache.srcReplicaName, snapshotName, dstRebuildingLvolAddress, mismatchingClusterList)
}

// RebuildingDstShallowCopyStart let the dst replica ask the src replica to start a shallow copy from a snapshot to the rebuilding lvol.
// Each time before starting a shallow copy, the dst replica will prepare a new rebuilding lvol and expose it as a NVMf bdev.
func (r *Replica) RebuildingDstShallowCopyStart(spdkClient *spdkclient.Client, snapshotName string, fastSync bool) (err error) {
	r.Lock()
	defer r.Unlock()

	defer func() {
		if err != nil {
			r.rebuildingDstCache.rebuildingError = err.Error()
			r.rebuildingDstCache.rebuildingState = types.ProgressStateError
			r.rebuildingDstCache.processingState = types.ProgressStateError
		}
	}()

	srcReplicaServiceCli, err := GetServiceClient(r.rebuildingDstCache.srcReplicaAddress)
	if err != nil {
		return err
	}
	defer func() {
		if errClose := srcReplicaServiceCli.Close(); errClose != nil {
			r.log.WithError(errClose).Errorf("Rebuilding dst replica failed to close src replica %s client with address %s during start rebuilding dst shallow copy", r.rebuildingDstCache.srcReplicaName, r.rebuildingDstCache.srcReplicaAddress)
		}
	}()

	dstRebuildingLvolAddress, requireRangeCopy, err := r.rebuildingDstShallowCopyPrepare(spdkClient, srcReplicaServiceCli, snapshotName, fastSync)
	if err != nil {
		return err
	}

	srcSnapSvcLvol := r.rebuildingDstCache.rebuildingSnapshotMap[snapshotName]
	if srcSnapSvcLvol == nil {
		return fmt.Errorf("cannot find snapshot %s in the rebuilding snapshot list for rebuilding dst replica %s shallow copy start", snapshotName, r.Name)
	}
	r.rebuildingDstCache.processingSnapshotName = snapshotName
	r.rebuildingDstCache.processingSize = 0
	r.rebuildingDstCache.processingState = types.ProgressStateInProgress
	r.rebuildingDstCache.snapshotTotalRebuildingSize = srcSnapSvcLvol.ActualSize

	dstSnapLvolName := GetReplicaSnapshotLvolName(r.Name, snapshotName)
	dstSnapBdevLvol, err := spdkClient.BdevLvolGetByName(spdktypes.GetLvolAlias(r.LvsName, dstSnapLvolName), 0)
	if err != nil {
		if !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return err
		}
		// Directly start a shallow copy when there is no existing snapshot lvol
		return srcReplicaServiceCli.ReplicaRebuildingSrcShallowCopyStart(r.rebuildingDstCache.srcReplicaName, snapshotName, dstRebuildingLvolAddress)
	}

	// Otherwise, try to reuse the existing lvol

	if requireRangeCopy {
		// Need to manually update the progress after reuse the intact existing snapshot lvol
		r.log.Infof("Rebuilding dst replica is starting range shallow copy for snapshot lvol %s", dstSnapLvolName)
		return r.rebuildingDstRangeShallowCopy(spdkClient, srcReplicaServiceCli, snapshotName, dstRebuildingLvolAddress)
	}

	r.log.Infof("Rebuilding dst replica directly reused an intact snapshot lvol %s then skipped the shallow copy", dstSnapLvolName)
	// Need to manually update the progress after reuse the existing snapshot lvol
	r.rebuildingDstCache.processingState = types.ProgressStateComplete
	r.rebuildingDstCache.processingSize = dstSnapBdevLvol.DriverSpecific.Lvol.NumAllocatedClusters * defaultClusterSize

	return nil
}

// RebuildingDstShallowCopyCheck let the dst replica ask the src replica to retrieve the shallow copy status based on the cached info.
func (r *Replica) RebuildingDstShallowCopyCheck(spdkClient *spdkclient.Client) (ret *spdkrpc.ReplicaRebuildingDstShallowCopyCheckResponse, err error) {
	r.Lock()
	defer r.Unlock()

	snapApiLvol := r.rebuildingDstCache.rebuildingSnapshotMap[r.rebuildingDstCache.processingSnapshotName]

	ret = &spdkrpc.ReplicaRebuildingDstShallowCopyCheckResponse{
		SrcReplicaName:    r.rebuildingDstCache.srcReplicaName,
		SrcReplicaAddress: r.rebuildingDstCache.srcReplicaAddress,
		SnapshotName:      r.rebuildingDstCache.processingSnapshotName,
	}

	// Allow checking the rebuilding record even if the rebuilding hasn't started or is already complete
	if !r.isRebuilding {
		ret.Error = r.rebuildingDstCache.rebuildingError
		ret.TotalState = r.rebuildingDstCache.rebuildingState
		ret.State = r.rebuildingDstCache.rebuildingState
		if r.rebuildingDstCache.rebuildingState == types.ProgressStateComplete {
			ret.Progress = 100
			ret.TotalProgress = 100
		} else {
			if snapApiLvol != nil && snapApiLvol.ActualSize != 0 {
				ret.Progress = uint32(float64(r.rebuildingDstCache.processingSize) / float64(snapApiLvol.ActualSize) * 100)
			}
			if r.rebuildingDstCache.rebuildingSize != 0 {
				ret.TotalProgress = uint32(float64(r.rebuildingDstCache.processingSize+r.rebuildingDstCache.processedSnapshotsSize) / float64(r.rebuildingDstCache.rebuildingSize) * 100)
			}
		}
		return ret, nil
	}

	// The dst replica has not started the shallow copy yet
	if r.rebuildingDstCache.processingState == types.ProgressStateStarting || r.rebuildingDstCache.processingSnapshotName == "" {
		return ret, nil
	}

	if snapApiLvol == nil {
		r.rebuildingDstCache.rebuildingError = fmt.Errorf("cannot find snapshot %s in the rebuilding snapshot list for shallow copy check", r.rebuildingDstCache.processingSnapshotName).Error()
		r.rebuildingDstCache.rebuildingState = types.ProgressStateError
		r.rebuildingDstCache.processingState = types.ProgressStateError
	}

	// If the processing shallow copy is already state complete or error, we cannot send the check request to the src replica again as spdk_tgt of the src replica has cleaned up the shallow copy op.
	if r.rebuildingDstCache.rebuildingState == types.ProgressStateInProgress &&
		r.rebuildingDstCache.processingState != types.ProgressStateComplete && r.rebuildingDstCache.processingState != types.ProgressStateError {
		srcReplicaServiceCli, err := GetServiceClient(r.rebuildingDstCache.srcReplicaAddress)
		if err != nil {
			return nil, err
		}
		defer func() {
			if errClose := srcReplicaServiceCli.Close(); errClose != nil {
				r.log.WithError(errClose).Errorf("Rebuilding dst replica failed to close src replica %s client with address %s during check rebuilding dst shallow copy", r.rebuildingDstCache.srcReplicaName, r.rebuildingDstCache.srcReplicaAddress)
			}
		}()

		state, handledClusters, totalClusters, errorMsg, err := srcReplicaServiceCli.ReplicaRebuildingSrcShallowCopyCheck(r.rebuildingDstCache.srcReplicaName, r.Name, r.rebuildingDstCache.processingSnapshotName)
		if err != nil {
			return nil, err
		}
		if errorMsg != "" {
			if r.rebuildingDstCache.rebuildingError == "" {
				r.rebuildingDstCache.rebuildingError = errorMsg
			}
			r.rebuildingDstCache.rebuildingState = types.ProgressStateError
			r.rebuildingDstCache.processingState = types.ProgressStateError
		} else {
			r.rebuildingDstCache.processingState = state
			r.rebuildingDstCache.processingSize = handledClusters * defaultClusterSize
			// After introducing range shallow copy, `totalClusters * defaultClusterSize` may be different from `snapApiLvol.ActualSize`
			// In this case, we need to correct `r.rebuildingDstCache.rebuildingSize`
			if r.rebuildingDstCache.snapshotTotalRebuildingSize != totalClusters*defaultClusterSize {
				r.rebuildingDstCache.snapshotTotalRebuildingSize = totalClusters * defaultClusterSize
				r.rebuildingDstCache.rebuildingSize = r.rebuildingDstCache.rebuildingSize - snapApiLvol.ActualSize + r.rebuildingDstCache.snapshotTotalRebuildingSize
				r.log.Infof("Rebuilding dst replica detected that snapshot %s shallow copy total size %d is different from the actual size %d, which typically means a range shallow copy", r.rebuildingDstCache.processingSnapshotName, r.rebuildingDstCache.snapshotTotalRebuildingSize, snapApiLvol.ActualSize)
			}
		}
	}

	if r.rebuildingDstCache.rebuildingError == "" {
		ret.State = r.rebuildingDstCache.processingState
		ret.TotalState = types.ProgressStateInProgress
		if r.rebuildingDstCache.snapshotTotalRebuildingSize == 0 {
			ret.Progress = 100
		} else {
			ret.Progress = uint32(float64(r.rebuildingDstCache.processingSize) / float64(r.rebuildingDstCache.snapshotTotalRebuildingSize) * 100)
		}
		if r.rebuildingDstCache.rebuildingSize == 0 {
			ret.TotalProgress = 100
		} else {
			ret.TotalProgress = uint32(float64(r.rebuildingDstCache.processingSize+r.rebuildingDstCache.processedSnapshotsSize) / float64(r.rebuildingDstCache.rebuildingSize) * 100)
		}
	} else {
		r.rebuildingDstCache.rebuildingState = types.ProgressStateError
		ret.Error = r.rebuildingDstCache.rebuildingError
		ret.State = types.InstanceStateError
		ret.TotalState = types.ProgressStateError
		if snapApiLvol == nil || snapApiLvol.ActualSize == 0 || r.rebuildingDstCache.snapshotTotalRebuildingSize == 0 {
			ret.Progress = 0
		} else {
			ret.Progress = uint32(float64(r.rebuildingDstCache.processingSize) / float64(r.rebuildingDstCache.snapshotTotalRebuildingSize) * 100)
		}
		if r.rebuildingDstCache.rebuildingSize == 0 {
			ret.TotalProgress = 0
		} else {
			ret.TotalProgress = uint32(float64(r.rebuildingDstCache.processingSize+r.rebuildingDstCache.processedSnapshotsSize) / float64(r.rebuildingDstCache.rebuildingSize) * 100)
		}
	}

	return ret, nil
}

// RebuildingDstSnapshotCreate creates a snapshot lvol based on the rebuilding lvol for the dst replica during the rebuilding process
func (r *Replica) RebuildingDstSnapshotCreate(spdkClient *spdkclient.Client, snapshotName string, opts *api.SnapshotOptions) (err error) {
	updateRequired := false

	r.Lock()
	defer func() {
		r.Unlock()

		if updateRequired {
			r.UpdateCh <- nil
		}
	}()

	if r.State != types.InstanceStateRunning {
		return fmt.Errorf("invalid state %v for dst replica %s rebuilding snapshot %s creation", r.State, r.Name, snapshotName)
	}
	if !r.isRebuilding {
		return fmt.Errorf("replica %s is not in rebuilding", r.Name)
	}
	if r.rebuildingDstCache.rebuildingLvol == nil {
		return fmt.Errorf("rebuilding lvol is not existed for dst replica %s rebuilding snapshot %s creation", r.Name, snapshotName)
	}

	defer func() {
		if err != nil {
			if r.State != types.InstanceStateError {
				r.State = types.InstanceStateError
				updateRequired = true
			}
			r.ErrorMsg = err.Error()
		} else {
			if r.State != types.InstanceStateError {
				r.ErrorMsg = ""
			}
		}
	}()

	srcSnapSvcLvol := r.rebuildingDstCache.rebuildingSnapshotMap[snapshotName]
	if srcSnapSvcLvol == nil {
		return fmt.Errorf("cannot find snapshot %s in the rebuilding snapshot list during dst replica %s rebuilding snapshot creation", snapshotName, r.Name)
	}
	// Guarantee the snapshot lvol has the correct parent after rebuilding
	dstSnapParentLvolName := ""
	if srcSnapSvcLvol.Parent == "" {
		if r.BackingImage != nil {
			dstSnapParentLvolName = r.BackingImage.Name
		}
	} else {
		dstSnapParentLvolName = GetReplicaSnapshotLvolName(r.Name, srcSnapSvcLvol.Parent)
	}

	var snapSvcLvol *Lvol
	snapLvolName := GetReplicaSnapshotLvolName(r.Name, snapshotName)
	snapBdevLvol, err := spdkClient.BdevLvolGetByName(spdktypes.GetLvolAlias(r.LvsName, snapLvolName), 0)
	if err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return err
	}
	if snapBdevLvol.UUID != "" { // If there is an existing snapshot lvol getting reused, check and correct its parent
		snapSvcLvol = BdevLvolInfoToServiceLvol(&snapBdevLvol)
		r.log.Infof("Rebuilding dst replica reused the intact existing snapshot %s(%s)", snapSvcLvol.Alias, snapSvcLvol.UUID)
	} else {
		// The snapshot lvol does not exist or the existing snapshot lvol is corrupted or outdated
		xattrs := getSnapshotXattrsFromOptions(opts)

		snapUUID, err := spdkClient.BdevLvolSnapshot(r.rebuildingDstCache.rebuildingLvol.UUID, snapLvolName, xattrs)
		if err != nil {
			return err
		}

		snapBdevLvol, err := spdkClient.BdevLvolGetByName(snapUUID, 0)
		if err != nil {
			return err
		}
		snapSvcLvol = BdevLvolInfoToServiceLvol(&snapBdevLvol)

		if r.rebuildingDstCache.rebuildingSnapshotMap[snapshotName] == nil {
			return fmt.Errorf("cannot find snapshot %s in the rebuilding snapshot list for rebuilding dst replica %s snapshot creation", snapshotName, r.Name)
		}
		if r.rebuildingDstCache.rebuildingSnapshotMap[snapshotName].ActualSize != snapSvcLvol.ActualSize {
			return fmt.Errorf("newly rebuilt snapshot %s(%s) actual size %d does not match the corresponding rebuilding src snapshot %s(%s) actual size %d during rebuilding dst replica %s snapshot creation", snapSvcLvol.Name, snapSvcLvol.UUID, snapSvcLvol.ActualSize, r.rebuildingDstCache.rebuildingSnapshotMap[snapshotName].Name, r.rebuildingDstCache.rebuildingSnapshotMap[snapshotName].UUID, r.rebuildingDstCache.rebuildingSnapshotMap[snapshotName].ActualSize, r.Name)
		}
		r.log.Infof("Rebuilding dst replica created a new snapshot %s(%s) with xattars %+v for rebuilding dst", snapSvcLvol.Alias, snapSvcLvol.UUID, xattrs)
	}

	if snapSvcLvol.Parent != dstSnapParentLvolName {
		// Corner case: the parent field of the src snapshot lvol will be empty even if its actual parent is the backing image
		if dstSnapParentLvolName == "" {
			if _, err := spdkClient.BdevLvolDetachParent(snapSvcLvol.Alias); err != nil {
				return err
			}
		} else { // The parent should be a regular snapshot lvol or the backing image
			if _, err := spdkClient.BdevLvolSetParent(snapSvcLvol.Alias, spdktypes.GetLvolAlias(r.LvsName, dstSnapParentLvolName)); err != nil {
				return err
			}
		}
		snapSvcLvol.Parent = dstSnapParentLvolName
		r.log.Infof("Rebuilding dst replica corrected the parent of the snapshot %s(%s) to %s for rebuilding dst replica snapshot creation", snapSvcLvol.Alias, snapSvcLvol.UUID, dstSnapParentLvolName)
	}

	// Blindly clean up the existing rebuilding lvol after each rebuilding dst replica snapshot creation
	rebuildingLvolName := GetReplicaRebuildingLvolName(r.Name)
	if r.rebuildingDstCache.rebuildingPort != 0 {
		r.log.Infof("Rebuilding dst replica is stopping exposing the rebuilding lvol %s before snapshot %s creation", rebuildingLvolName, snapshotName)
		if err := spdkClient.StopExposeBdev(helpertypes.GetNQN(rebuildingLvolName)); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return err
		}
	}
	if r.rebuildingDstCache.rebuildingLvol != nil {
		if _, err := spdkClient.BdevLvolDelete(r.rebuildingDstCache.rebuildingLvol.UUID); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			return err
		}
		r.rebuildingDstCache.rebuildingLvol = nil
	}

	// Do not update r.ActiveChain for the rebuilding snapshots here.
	// The replica will directly reconstruct r.ActiveChain as well as r.SnapshotLvolMap during the rebuilding dst finish.
	r.rebuildingDstCache.processedSnapshotList = append(r.rebuildingDstCache.processedSnapshotList, snapshotName)
	r.rebuildingDstCache.processedSnapshotsSize += snapSvcLvol.ActualSize

	r.rebuildingDstCache.processingState = types.ProgressStateStarting
	r.rebuildingDstCache.processingSnapshotName = ""
	r.rebuildingDstCache.processingSize = 0
	updateRequired = true

	return nil
}

// RebuildingDstSetQos sets a write bandwidth QoS limit on the rebuilding Lvol
// for the destination replica during the shallow copy process.
func (r *Replica) RebuildingDstSetQos(spdkClient *spdkclient.Client, qosLimitMbps int64) error {
	r.Lock()
	defer r.Unlock()

	// Store the QoS limit that will be applied to each rebuilding lvol
	r.rebuildingQosLimitMbps = qosLimitMbps

	if r.State != types.InstanceStateRunning {
		return fmt.Errorf("invalid state %v for dst replica %s to set QoS", r.State, r.Name)
	}
	if !r.isRebuilding {
		return fmt.Errorf("replica %s is not in rebuilding, cannot apply QoS", r.Name)
	}
	if r.rebuildingDstCache.rebuildingLvol == nil {
		return fmt.Errorf("rebuilding lvol does not exist for replica %s", r.Name)
	}

	lvolUUID := r.rebuildingDstCache.rebuildingLvol.UUID
	if lvolUUID == "" {
		return fmt.Errorf("rebuilding lvol UUID is empty for replica %s", r.Name)
	}

	// Apply write bandwidth QoS (MB/s)
	if err := spdkClient.BdevSetQosLimit(lvolUUID, 0, 0, 0, qosLimitMbps); err != nil {
		return fmt.Errorf("failed to set QoS limit %d MB/s on replica %s lvol %s: %v", qosLimitMbps, r.Name, lvolUUID, err)
	}

	r.log.Infof("Rebuilding dst replica applied QoS limit %d MB/s to rebuilding lvol %s(%s)", qosLimitMbps, r.rebuildingDstCache.rebuildingLvol.Alias, lvolUUID)
	return nil
}

func (r *Replica) BackupRestore(spdkClient *spdkclient.Client, backupUrl, snapshotName string, credential map[string]string, concurrentLimit int32) (err error) {
	r.Lock()
	defer r.Unlock()

	defer func() {
		if err == nil {
			r.isRestoring = true
		}
	}()

	if r.isRestoring {
		return fmt.Errorf("cannot initiate backup restore as there is one already in progress")
	}

	backupType, err := butil.CheckBackupType(backupUrl)
	if err != nil {
		err = errors.Wrapf(err, "failed to check the type for restoring backup %v", backupUrl)
		return grpcstatus.Errorf(grpccodes.InvalidArgument, "%v", err)
	}

	err = butil.SetupCredential(backupType, credential)
	if err != nil {
		err = errors.Wrapf(err, "failed to setup credential for restoring backup %v", backupUrl)
		return grpcstatus.Errorf(grpccodes.Internal, "%v", err)
	}

	backupName, _, _, err := backupstore.DecodeBackupURL(util.UnescapeURL(backupUrl))
	if err != nil {
		err = errors.Wrapf(err, "failed to decode backup url %v", backupUrl)
		return grpcstatus.Errorf(grpccodes.InvalidArgument, "%v", err)
	}

	if r.restore == nil {
		return grpcstatus.Errorf(grpccodes.NotFound, "restoration for backup %v is not initialized", backupUrl)
	}

	restore := r.restore.DeepCopy()
	if restore.State == btypes.ProgressStateError {
		return fmt.Errorf("cannot start restoring backup %v of the previous failed restoration", backupUrl)
	}

	if restore.LastRestored == backupName {
		return grpcstatus.Errorf(grpccodes.AlreadyExists, "already restored backup %v", backupName)
	}

	// Initialize `r.restore`
	// First restore request. It must be a normal full restore.
	if restore.LastRestored == "" && (restore.State == btypes.ProgressStateUndefined || restore.State == btypes.ProgressStateCanceled) {
		r.log.Infof("Starting a new restore for backup %v with restore state %v", backupUrl, restore.State)
		lvolName := GetReplicaSnapshotLvolName(r.Name, snapshotName)
		r.restore, err = NewRestore(spdkClient, lvolName, snapshotName, backupUrl, backupName, r)
		if err != nil {
			err = errors.Wrap(err, "failed to start new restore")
			return grpcstatus.Errorf(grpccodes.Internal, "%v", err)
		}
	} else {
		r.log.Infof("Resetting the restore for backup %v", backupUrl)

		var lvolName string
		var snapshotNameToBeRestored string

		validLastRestoredBackup := r.canDoIncrementalRestore(restore, backupUrl, backupName)
		if validLastRestoredBackup {
			r.log.Infof("Starting an incremental restore for backup %v", backupUrl)
		} else {
			r.log.Infof("Starting a full restore for backup %v", backupUrl)
		}

		lvolName = GetReplicaSnapshotLvolName(r.Name, snapshotName)
		snapshotNameToBeRestored = snapshotName

		r.restore.StartNewRestore(backupUrl, backupName, lvolName, snapshotNameToBeRestored, validLastRestoredBackup)
	}

	// Initiate restore
	newRestore := r.restore.DeepCopy()
	defer func() {
		if err != nil { // nolint:staticcheck
			// TODO: Support snapshot revert for incremental restore
			r.log.WithError(err).Error("Failed to start backup restore")
		}
	}()

	isFullRestore := newRestore.LastRestored == ""

	defer func() {
		go func() {
			if err := r.completeBackupRestore(spdkClient, isFullRestore); err != nil {
				r.log.WithError(err).Warn("Replica failed to complete backup restore")
			}
		}()
	}()

	if isFullRestore {
		r.log.Infof("Starting a new full restore for backup %v", backupUrl)
		if err := r.backupRestore(backupUrl, newRestore.LvolName, concurrentLimit); err != nil {
			return errors.Wrapf(err, "failed to start full backup restore")
		}
		r.log.Infof("Successfully initiated full restore for %v to %v", backupUrl, newRestore.LvolName)
	} else {
		r.log.Infof("Starting an incremental restore for backup %v", backupUrl)
		if err := r.backupRestoreIncrementally(backupUrl, newRestore.LastRestored, newRestore.LvolName, concurrentLimit); err != nil {
			return errors.Wrapf(err, "failed to start incremental backup restore")
		}
		r.log.Infof("Successfully initiated incremental restore for %v to %v", backupUrl, newRestore.LvolName)
	}

	return nil

}

func (r *Replica) backupRestoreIncrementally(backupURL, lastRestored, snapshotLvolName string, concurrentLimit int32) error {
	backupURL = butil.UnescapeURL(backupURL)

	r.log.WithFields(logrus.Fields{
		"backupURL":        backupURL,
		"lastRestored":     lastRestored,
		"snapshotLvolName": snapshotLvolName,
		"concurrentLimit":  concurrentLimit,
	}).Info("Start restoring backup incrementally")

	return backupstore.RestoreDeltaBlockBackupIncrementally(r.ctx, &backupstore.DeltaRestoreConfig{
		BackupURL:       backupURL,
		DeltaOps:        r.restore,
		LastBackupName:  lastRestored,
		Filename:        snapshotLvolName,
		ConcurrentLimit: int32(concurrentLimit),
	})
}

func (r *Replica) backupRestore(backupURL, snapshotLvolName string, concurrentLimit int32) error {
	backupURL = butil.UnescapeURL(backupURL)

	r.log.WithFields(logrus.Fields{
		"backupURL":        backupURL,
		"snapshotLvolName": snapshotLvolName,
		"concurrentLimit":  concurrentLimit,
	}).Info("Start restoring backup")

	return backupstore.RestoreDeltaBlockBackup(r.ctx, &backupstore.DeltaRestoreConfig{
		BackupURL:       backupURL,
		DeltaOps:        r.restore,
		Filename:        snapshotLvolName,
		ConcurrentLimit: int32(concurrentLimit),
	})
}

func (r *Replica) canDoIncrementalRestore(restore *Restore, backupURL, requestedBackupName string) bool {
	if restore.LastRestored == "" {
		r.log.Warnf("There is a restore record in the server but last restored backup is empty with restore state is %v, will do full restore instead", restore.State)
		return false
	}
	if _, err := backupstore.InspectBackup(strings.Replace(backupURL, requestedBackupName, restore.LastRestored, 1)); err != nil {
		r.log.WithError(err).Warnf("The last restored backup %v becomes invalid for incremental restore, will do full restore instead", restore.LastRestored)
		return false
	}
	return true
}

func (r *Replica) completeBackupRestore(spdkClient *spdkclient.Client, isFullRestore bool) (err error) {
	defer func() {
		if extraErr := r.finishRestore(err); extraErr != nil {
			r.log.WithError(extraErr).Error("Failed to finish backup restore")
		}
	}()

	if err := r.waitForRestoreComplete(); err != nil {
		return errors.Wrapf(err, "failed to wait for restore complete")
	}

	r.RLock()
	restore := r.restore.DeepCopy()
	r.RUnlock()

	if isFullRestore {
		return r.postFullRestoreOperations(spdkClient, restore)
	}

	return r.postIncrementalRestoreOperations(spdkClient, restore)
}

func (r *Replica) waitForRestoreComplete() error {
	periodicChecker := time.NewTicker(time.Duration(restorePeriodicRefreshInterval.Seconds()) * time.Second)
	defer periodicChecker.Stop()

	for range periodicChecker.C {
		r.restore.RLock()
		restoreProgress := r.restore.Progress
		restoreError := r.restore.Error
		restoreState := r.restore.State
		r.restore.RUnlock()

		if restoreProgress == 100 {
			r.log.Info("Backup restoration completed successfully")
			return nil
		}
		if restoreState == btypes.ProgressStateCanceled {
			r.log.Info("Backup restoration is cancelled")
			return nil
		}
		if restoreError != "" {
			err := fmt.Errorf("%v", restoreError)
			r.log.WithError(err).Errorf("Found backup restoration error")
			return err
		}
	}
	return nil
}

func (r *Replica) postIncrementalRestoreOperations(spdkClient *spdkclient.Client, restore *Restore) error {
	r.log.Infof("Replacing snapshot %v of the restored volume", restore.SnapshotName)

	if r.restore.State == btypes.ProgressStateCanceled {
		r.log.Info("Doing nothing for canceled backup restoration")
		return nil
	}

	// Delete snapshot; SPDK will coalesce the content into the current head lvol.
	r.log.Infof("Deleting snapshot %v for snapshot replacement of the restored volume", restore.SnapshotName)
	_, err := r.SnapshotDelete(spdkClient, restore.SnapshotName)
	if err != nil {
		r.log.WithError(err).Error("Failed to delete snapshot of the restored volume")
		return errors.Wrapf(err, "failed to delete snapshot of the restored volume")
	}

	r.log.Infof("Creating snapshot %v for snapshot replacement of the restored volume", restore.SnapshotName)
	opts := &api.SnapshotOptions{
		UserCreated: false,
		Timestamp:   util.Now(),
	}
	_, err = r.SnapshotCreate(spdkClient, restore.SnapshotName, opts)
	if err != nil {
		r.log.WithError(err).Error("Failed to take snapshot of the restored volume")
		return errors.Wrapf(err, "failed to take snapshot of the restored volume")
	}

	r.log.Infof("Done running incremental restore %v to lvol %v", restore.BackupURL, restore.LvolName)
	return nil
}

func (r *Replica) postFullRestoreOperations(spdkClient *spdkclient.Client, restore *Restore) error {
	if r.restore.State == btypes.ProgressStateCanceled {
		r.log.Info("Doing nothing for canceled backup restoration")
		return nil
	}

	snapLvolName := GetReplicaSnapshotLvolName(r.Name, restore.SnapshotName)
	if _, exists := r.SnapshotLvolMap[snapLvolName]; exists {
		r.log.Infof("Deleting existing snapshot %v of the restored volume", snapLvolName)
		_, err := r.SnapshotDelete(spdkClient, restore.SnapshotName)
		if err != nil {
			r.log.WithError(err).Errorf("Failed to delete existing snapshot %v of the restored volume", snapLvolName)
			return errors.Wrapf(err, "failed to delete snapshot %v of the restored volume", snapLvolName)
		}
	}

	r.log.Infof("Taking snapshot %v of the restored volume", restore.SnapshotName)
	opts := &api.SnapshotOptions{
		UserCreated: false,
		Timestamp:   util.Now(),
	}
	_, err := r.SnapshotCreate(spdkClient, restore.SnapshotName, opts)
	if err != nil {
		r.log.WithError(err).Error("Failed to take snapshot of the restored volume")
		return errors.Wrapf(err, "failed to take snapshot of the restored volume")
	}

	r.log.Infof("Done running full restore %v to lvol %v (snapshot %v)", restore.BackupURL, restore.LvolName, restore.SnapshotName)
	return nil
}

func (r *Replica) finishRestore(restoreErr error) error {
	r.Lock()
	defer r.Unlock()

	defer func() {
		if r.restore == nil {
			return
		}
		if restoreErr != nil {
			r.restore.UpdateRestoreStatus(r.restore.LvolName, 0, restoreErr)
			return
		}
		r.restore.FinishRestore()
	}()

	if !r.isRestoring {
		err := fmt.Errorf("BUG: volume is not being restored")
		if restoreErr != nil {
			restoreErr = util.CombineErrors(err, restoreErr)
		} else {
			restoreErr = err
		}
		return err
	}

	r.log.Infof("Unflagging isRestoring")
	r.isRestoring = false

	return nil
}

func (r *Replica) SetErrorState() {
	needUpdate := false

	r.Lock()
	defer func() {
		r.Unlock()

		if needUpdate {
			r.UpdateCh <- nil
		}
	}()

	if r.State != types.InstanceStateStopped && r.State != types.InstanceStateError {
		r.State = types.InstanceStateError
		needUpdate = true
	}
}

// CleanupLvolTree retrieves the lvol tree with BFS. Then try its best effort to do cleanup bottom up.
func (r *Replica) CleanupLvolTree(spdkClient *spdkclient.Client, rootLvolName string, bdevLvolMap map[string]*spdktypes.BdevInfo) {
	var queue []*spdktypes.BdevInfo
	if bdevLvolMap[rootLvolName] != nil {
		queue = []*spdktypes.BdevInfo{bdevLvolMap[rootLvolName]}
	}
	for idx := 0; idx < len(queue); idx++ {
		for _, childLvolName := range queue[idx].DriverSpecific.Lvol.Clones {
			if bdevLvolMap[childLvolName] != nil {
				queue = append(queue, bdevLvolMap[childLvolName])
			}
		}
	}
	for idx := len(queue) - 1; idx >= 0; idx-- {
		// This may fail since there may be a rebuilding failed replicas on the same host that leaves an orphan rebuilding lvol as a child of a snapshot lvol.
		// Then this snapshot lvol would have multiple children then cannot be deleted.
		if _, err := spdkClient.BdevLvolDelete(queue[idx].UUID); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
			r.log.WithError(err).Errorf("Failed to delete lvol %v(%v) from the lvol tree with root %v(%s), this lvol may accidentally have some leftover orphans children %+v, will continue", queue[idx].Aliases[0], queue[idx].UUID, bdevLvolMap[rootLvolName].Aliases[0], bdevLvolMap[rootLvolName].UUID, queue[idx].DriverSpecific.Lvol.Clones)
		}
	}
}

// SetTestErrorState sets the replica's internal state to Error for testing purposes.
// This simulates what happens during a real production shallow copy failure,
// where RebuildingDstShallowCopyStart's defer sets r.State = InstanceStateError.
// Test hooks operate at the Engine level and don't trigger this per-replica state change,
// so this method allows tests to simulate the production error path.
func (r *Replica) SetTestErrorState(errMsg string) {
	r.Lock()
	defer r.Unlock()
	r.State = types.InstanceStateError
	r.ErrorMsg = errMsg
	if r.rebuildingDstCache.rebuildingError == "" {
		r.rebuildingDstCache.rebuildingError = errMsg
		r.rebuildingDstCache.rebuildingState = types.ProgressStateError
	}
}
</file>

<file path="pkg/spdk/restore.go">
package spdk

import (
	"fmt"
	"os"
	"strconv"
	"strings"
	"sync"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"

	"github.com/longhorn/backupstore"

	"github.com/longhorn/go-spdk-helper/pkg/initiator"

	btypes "github.com/longhorn/backupstore/types"
	commonns "github.com/longhorn/go-common-libs/ns"
	commontypes "github.com/longhorn/go-common-libs/types"
	spdkclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
	helpertypes "github.com/longhorn/go-spdk-helper/pkg/types"
	helperutil "github.com/longhorn/go-spdk-helper/pkg/util"
)

type Restore struct {
	sync.RWMutex

	spdkClient *spdkclient.Client
	replica    *Replica

	Progress  int
	Error     string
	BackupURL string
	State     btypes.ProgressState

	// The snapshot file that stores the restored data in the end.
	LvolName     string
	SnapshotName string

	LastRestored           string
	CurrentRestoringBackup string

	ip             string
	port           int32
	executor       *commonns.Executor
	subsystemNQN   string
	controllerName string
	initiator      *initiator.Initiator

	stopOnce sync.Once
	stopChan chan struct{}

	log logrus.FieldLogger
}

var _ backupstore.DeltaRestoreOperations = (*Restore)(nil)

func NewRestore(spdkClient *spdkclient.Client, lvolName, snapshotName, backupUrl, backupName string, replica *Replica) (*Restore, error) {
	log := logrus.WithFields(logrus.Fields{
		"lvolName":     lvolName,
		"snapshotName": snapshotName,
		"backupUrl":    backupUrl,
		"backupName":   backupName,
	})

	executor, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	if err != nil {
		return nil, errors.Wrapf(err, "failed to create executor")
	}

	return &Restore{
		spdkClient:             spdkClient,
		replica:                replica,
		BackupURL:              backupUrl,
		CurrentRestoringBackup: backupName,
		LvolName:               lvolName,
		SnapshotName:           snapshotName,
		ip:                     replica.IP,
		port:                   replica.PortStart,
		executor:               executor,
		State:                  btypes.ProgressStateInProgress,
		Progress:               0,
		stopChan:               make(chan struct{}),
		log:                    log,
	}, nil
}

func (r *Restore) StartNewRestore(backupUrl, currentRestoringBackup, lvolName, snapshotName string, validLastRestoredBackup bool) {
	r.Lock()
	defer r.Unlock()

	r.LvolName = lvolName
	r.SnapshotName = snapshotName

	r.Progress = 0
	r.Error = ""
	r.BackupURL = backupUrl
	r.State = btypes.ProgressStateInProgress
	if !validLastRestoredBackup {
		r.LastRestored = ""
	}
	r.CurrentRestoringBackup = currentRestoringBackup
}

func (r *Restore) DeepCopy() *Restore {
	r.RLock()
	defer r.RUnlock()

	return &Restore{
		LvolName:               r.LvolName,
		SnapshotName:           r.SnapshotName,
		LastRestored:           r.LastRestored,
		BackupURL:              r.BackupURL,
		CurrentRestoringBackup: r.CurrentRestoringBackup,
		State:                  r.State,
		Error:                  r.Error,
		Progress:               r.Progress,
	}
}

func (r *Restore) OpenVolumeDev(volDevName string) (*os.File, string, error) {
	lvolName := r.replica.Name

	r.log.Info("Unexposing lvol bdev before restoration")
	if r.replica.IsExposed {
		err := r.spdkClient.StopExposeBdev(helpertypes.GetNQN(lvolName))
		if err != nil {
			return nil, "", errors.Wrapf(err, "failed to unexpose lvol bdev %v", lvolName)
		}
		r.replica.IsExposed = false
	}

	r.log.Info("Exposing snapshot lvol bdev for restore")
	subsystemNQN, controllerName, err := exposeSnapshotLvolBdev(r.spdkClient, r.replica.LvsName, lvolName, r.ip, r.port, r.executor)
	if err != nil {
		r.log.WithError(err).Errorf("Failed to expose lvol bdev")
		return nil, "", err
	}
	r.subsystemNQN = subsystemNQN
	r.controllerName = controllerName
	r.replica.IsExposed = true
	r.log.Infof("Exposed snapshot lvol bdev %v, subsystemNQN=%v, controllerName %v", lvolName, subsystemNQN, controllerName)

	r.log.Info("Creating NVMe initiator for lvol bdev")
	nvmeTCPInfo := &initiator.NVMeTCPInfo{
		SubsystemNQN: helpertypes.GetNQN(lvolName),
	}
	i, err := initiator.NewInitiator(lvolName, initiator.HostProc, nvmeTCPInfo, nil)
	if err != nil {
		return nil, "", errors.Wrapf(err, "failed to create NVMe initiator for lvol bdev %v", lvolName)
	}
	if _, err := i.StartNvmeTCPInitiator(r.ip, strconv.Itoa(int(r.port)), true, true); err != nil {
		return nil, "", errors.Wrapf(err, "failed to start NVMe initiator for lvol bdev %v", lvolName)
	}
	r.initiator = i

	r.log.Infof("Opening NVMe device %v", r.initiator.Endpoint)
	fh, err := os.OpenFile(r.initiator.Endpoint, os.O_RDONLY, 0666)
	if err != nil {
		return nil, "", errors.Wrapf(err, "failed to open NVMe device %v for lvol bdev %v", r.initiator.Endpoint, lvolName)
	}

	return fh, r.initiator.Endpoint, err
}

func (r *Restore) CloseVolumeDev(volDev *os.File) error {
	r.log.Infof("Closing NVMe device %v", r.initiator.Endpoint)
	if err := volDev.Close(); err != nil {
		return errors.Wrapf(err, "failed to close NVMe device %v", r.initiator.Endpoint)
	}

	r.log.Info("Stopping NVMe initiator")
	if _, err := r.initiator.Stop(nil, true, true, false); err != nil {
		return errors.Wrapf(err, "failed to stop NVMe initiator")
	}

	if !r.replica.IsExposed {
		r.log.Info("Unexposing lvol bdev")
		lvolName := r.replica.Name
		err := r.spdkClient.StopExposeBdev(helpertypes.GetNQN(lvolName))
		if err != nil {
			return errors.Wrapf(err, "failed to unexpose lvol bdev %v", lvolName)
		}
		r.replica.IsExposed = false
	}

	return nil
}

func (r *Restore) UpdateRestoreStatus(snapshotLvolName string, progress int, err error) {
	r.Lock()
	defer r.Unlock()

	r.LvolName = snapshotLvolName
	r.Progress = progress

	if err != nil {
		r.CurrentRestoringBackup = ""

		// No need to mark restore as error if it's cancelled.
		// The restoration will be restarted after the engine is restarted.
		if strings.Contains(err.Error(), btypes.ErrorMsgRestoreCancelled) {
			r.log.WithError(err).Warn("Backup restoration is cancelled")
			r.State = btypes.ProgressStateCanceled
		} else {
			r.log.WithError(err).Error("Backup restoration is failed")
			r.State = btypes.ProgressStateError
			if r.Error != "" {
				r.Error = fmt.Sprintf("%v: %v", err.Error(), r.Error)
			} else {
				r.Error = err.Error()
			}
		}
	}
}

func (r *Restore) FinishRestore() {
	r.Lock()
	defer r.Unlock()

	if r.State != btypes.ProgressStateError && r.State != btypes.ProgressStateCanceled {
		r.State = btypes.ProgressStateComplete
		r.LastRestored = r.CurrentRestoringBackup
		r.CurrentRestoringBackup = ""
	}
}

func (r *Restore) Stop() {
	r.stopOnce.Do(func() {
		close(r.stopChan)
	})
}

func (r *Restore) GetStopChan() chan struct{} {
	return r.stopChan
}
</file>

<file path="pkg/spdk/server_backingimage.go">
package spdk

import (
	"context"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"

	"google.golang.org/protobuf/types/known/emptypb"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/types/pkg/generated/spdkrpc"

	spdktypes "github.com/longhorn/go-spdk-helper/pkg/spdk/types"

	"github.com/longhorn/longhorn-spdk-engine/pkg/types"
)

// BackingImageGet will return the backing image information in the server.
func (s *Server) BackingImageCreate(ctx context.Context, req *spdkrpc.BackingImageCreateRequest) (ret *spdkrpc.BackingImage, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "backing image name is required")
	}
	if req.BackingImageUuid == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "backing image UUID is required")
	}
	if req.Size == uint64(0) {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "backing image size is required")
	}
	if req.LvsUuid == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "lvs UUID is required")
	}
	if req.Checksum == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "checksum is required")
	}

	// Don't recreate the backing image
	backingImageSnapLvolName := GetBackingImageSnapLvolName(req.Name, req.LvsUuid)

	s.RLock()
	bi := s.backingImageMap[backingImageSnapLvolName]
	spdkClient := s.spdkClient
	s.RUnlock()

	if bi != nil {
		if bi.BackingImageUUID == req.BackingImageUuid {
			return nil, grpcstatus.Errorf(grpccodes.AlreadyExists, "backing image %v already exists", req.Name)
		}

		logrus.Infof("Found backing image exists with different backing image UUID %v, deleting it", bi.BackingImageUUID)

		if err := bi.Delete(spdkClient, s.portAllocator); err != nil {
			return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to delete backing image %v in lvs %v with different UUID", req.Name, req.LvsUuid).Error())
		}

		s.Lock()
		delete(s.backingImageMap, backingImageSnapLvolName)
		s.Unlock()
	}

	newBI, err := s.newBackingImage(req)
	if err != nil {
		return nil, err
	}

	s.RLock()
	spdkClient = s.spdkClient
	s.RUnlock()

	return newBI.Create(spdkClient, s.portAllocator, req.FromAddress, req.SrcLvsUuid)
}

// BackingImageDelete will delete the backing image.
func (s *Server) BackingImageDelete(ctx context.Context, req *spdkrpc.BackingImageDeleteRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "backing image name is required")
	}
	if req.LvsUuid == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "lvs UUID is required")
	}

	s.RLock()
	bi := s.backingImageMap[GetBackingImageSnapLvolName(req.Name, req.LvsUuid)]
	spdkClient := s.spdkClient
	s.RUnlock()

	defer func() {
		if err == nil {
			s.Lock()
			delete(s.backingImageMap, GetBackingImageSnapLvolName(req.Name, req.LvsUuid))
			s.Unlock()
		}
	}()

	if bi != nil {
		if err := bi.Delete(spdkClient, s.portAllocator); err != nil {
			return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to delete backing image %v in lvs %v", req.Name, req.LvsUuid).Error())
		}
	}

	return &emptypb.Empty{}, nil
}

// BackingImageGet will return the backing image information in the server.
func (s *Server) BackingImageGet(ctx context.Context, req *spdkrpc.BackingImageGetRequest) (ret *spdkrpc.BackingImage, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "backing image name is required")
	}
	if req.LvsUuid == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "lvs UUID is required")
	}

	backingImageSnapLvolName := GetBackingImageSnapLvolName(req.Name, req.LvsUuid)

	s.RLock()
	bi := s.backingImageMap[backingImageSnapLvolName]
	spdkClient := s.spdkClient
	s.RUnlock()

	if bi == nil {
		lvsName, err := GetLvsNameByUUID(spdkClient, req.LvsUuid)
		if err != nil {
			return nil, grpcstatus.Errorf(grpccodes.NotFound, "failed to get the lvs name with lvs uuid %v", req.LvsUuid)
		}

		if lvsName != "" {
			backingImageSnapLvolAlias := spdktypes.GetLvolAlias(lvsName, backingImageSnapLvolName)
			bdevLvolList, err := spdkClient.BdevLvolGet(backingImageSnapLvolAlias, 0)
			if err != nil {
				return nil, grpcstatus.Errorf(grpccodes.NotFound, "got error %v when getting lvol %v in the lvs %v", err, req.Name, req.LvsUuid)
			}
			if len(bdevLvolList) != 1 {
				return nil, grpcstatus.Errorf(grpccodes.NotFound, "zero or multiple lvols with alias %s found when finding backing image %v in lvs %v", backingImageSnapLvolAlias, req.Name, req.LvsUuid)
			}
			// If we can get the lvol, verify() will reconstruct the backing image record in the server, should inform the caller
			return nil, grpcstatus.Errorf(grpccodes.NotFound, "backing image %v lvol found in the lvs %v but failed to find the record in the server", req.Name, req.LvsUuid)
		}
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find backing image %v in lvs %v", req.Name, req.LvsUuid)
	}

	return bi.Get(), nil
}

// BackingImageList will return the backing image list in the server.
func (s *Server) BackingImageList(ctx context.Context, req *emptypb.Empty) (ret *spdkrpc.BackingImageListResponse, err error) {
	backingImageMap := map[string]*BackingImage{}
	res := map[string]*spdkrpc.BackingImage{}

	s.RLock()
	for k, v := range s.backingImageMap {
		backingImageMap[k] = v
	}
	s.RUnlock()

	// backingImageName is in the form of "bi-%s-disk-%s"
	for backingImageName, bi := range backingImageMap {
		res[backingImageName] = bi.Get()
	}

	return &spdkrpc.BackingImageListResponse{BackingImages: res}, nil
}

// BackingImageWatch will watch the backing image update.
func (s *Server) BackingImageWatch(req *emptypb.Empty, srv spdkrpc.SPDKService_BackingImageWatchServer) error {
	responseCh, err := s.Subscribe(types.InstanceTypeBackingImage)
	if err != nil {
		return err
	}

	defer func() {
		if err != nil {
			logrus.WithError(err).Error("SPDK service backing image watch errored out")
		} else {
			logrus.Info("SPDK service backing image watch ended successfully")
		}
	}()
	logrus.Info("Started new SPDK service backing image update watch")

	done := false
	for {
		select {
		case <-s.ctx.Done():
			logrus.Info("spdk gRPC server: stopped backing image watch due to the context done")
			done = true
		case <-responseCh:
			if err := srv.Send(&emptypb.Empty{}); err != nil {
				return err
			}
		}
		if done {
			break
		}
	}

	return nil
}

// BackingImageExpose will expose the backing image as a new lvol.
func (s *Server) BackingImageExpose(ctx context.Context, req *spdkrpc.BackingImageGetRequest) (ret *spdkrpc.BackingImageExposeResponse, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "backing image name is required")
	}
	if req.LvsUuid == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "lvs UUID is required")
	}
	s.RLock()
	bi := s.backingImageMap[GetBackingImageSnapLvolName(req.Name, req.LvsUuid)]
	spdkClient := s.spdkClient
	s.RUnlock()

	if bi == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find backing image %v in lvs %v", req.Name, req.LvsUuid)
	}

	exposedSnapshotLvolAddress, err := bi.BackingImageExpose(spdkClient, s.portAllocator)
	if err != nil {
		return nil, err
	}
	return &spdkrpc.BackingImageExposeResponse{ExposedSnapshotLvolAddress: exposedSnapshotLvolAddress}, nil

}

// BackingImageUnexpose will unexpose the backing image.
func (s *Server) BackingImageUnexpose(ctx context.Context, req *spdkrpc.BackingImageGetRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "backing image name is required")
	}
	if req.LvsUuid == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "lvs UUID is required")
	}
	s.RLock()
	bi := s.backingImageMap[GetBackingImageSnapLvolName(req.Name, req.LvsUuid)]
	spdkClient := s.spdkClient
	s.RUnlock()

	if bi == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find backing image %v in lvs %v", req.Name, req.LvsUuid)
	}

	err = bi.BackingImageUnexpose(spdkClient, s.portAllocator)
	if err != nil {
		return nil, grpcstatus.Error(grpccodes.Internal, errors.Wrapf(err, "failed to unexpose backing image %v in lvs %v", req.Name, req.LvsUuid).Error())
	}
	return &emptypb.Empty{}, nil
}
</file>

<file path="pkg/spdk/server_disk.go">
package spdk

import (
	"context"

	"github.com/sirupsen/logrus"

	"google.golang.org/protobuf/types/known/emptypb"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/types/pkg/generated/spdkrpc"
)

func (s *Server) DiskCreate(ctx context.Context, req *spdkrpc.DiskCreateRequest) (*spdkrpc.Disk, error) {
	s.Lock()
	spdkClient := s.spdkClient

	disk, exists := s.diskMap[req.DiskName]
	if exists {
		if disk.GetState() == DiskStateReady {
			s.Unlock()
			return disk.DiskGet(spdkClient, req.DiskName, req.DiskPath, req.DiskDriver)
		}
		s.Unlock()

		return &spdkrpc.Disk{
			State: string(disk.GetState()),
		}, nil
	}

	disk = NewDisk(req.DiskName, req.DiskUuid, req.DiskPath, req.DiskDriver, req.BlockSize)
	s.diskMap[req.DiskName] = disk
	s.Unlock()

	go func(d *Disk, req *spdkrpc.DiskCreateRequest) {
		// Serialize SPDK disk creation to avoid race conditions in global SPDK subsystems
		// The upper-level DiskCreate() flow remains asynchronous — the gRPC call immediately returns and the creation
		// runs in a background goroutine — but within that goroutine, we serialize the
		// lower-level SPDK operations to ensure controller attach and lvstore operations
		// are performed safely and deterministically without concurrent access issues.

		// It may have issues if multiple disks are being created simultaneously without this lock
		s.diskCreateLock.Lock()
		defer s.diskCreateLock.Unlock()

		// Disabling hotplug is a best-effort guard to improve stability; creation continues even if this call fails.
		// TODO: If virtio-blk requires the same handling, add similar guards to the virtio-blk path.
		if isNvmeDriver(req.DiskDriver, req.DiskPath) {
			_ = setNvmeHotPlug(spdkClient, false)
			defer func() {
				if success := setNvmeHotPlug(spdkClient, true); success {
					s.hotplugActive.Store(true)
				} else {
					s.hotplugActive.Store(false)
				}
			}()
		}

		if err := d.DiskCreate(spdkClient, req.DiskName, req.DiskUuid, req.DiskPath, req.DiskDriver, req.BlockSize); err != nil {
			logrus.WithError(err).Errorf("Failed to create disk %s(%s) path %s", req.DiskName, req.DiskUuid, req.DiskPath)
			return
		}

		logrus.Infof("Disk %v is created, replicas will be discovered by the next monitoring cycle", req.DiskName)
	}(disk, req)

	return &spdkrpc.Disk{
		State: string(disk.GetState()),
	}, nil
}

func (s *Server) DiskDelete(ctx context.Context, req *spdkrpc.DiskDeleteRequest) (ret *emptypb.Empty, err error) {
	s.RLock()
	disk := s.diskMap[req.DiskName]
	spdkClient := s.spdkClient
	s.RUnlock()

	defer func() {
		if err == nil {
			s.Lock()
			delete(s.diskMap, req.DiskName)
			s.Unlock()
		}
	}()

	if disk == nil {
		// If the specified disk does not exist, tlog a warning for visibility.
		logrus.Warnf("Disk %s not found; skipping deletion", req.DiskName)
		return &emptypb.Empty{}, nil
	}

	return disk.DiskDelete(spdkClient, req.DiskName, req.DiskUuid, req.DiskPath, req.DiskDriver)
}

func (s *Server) DiskGet(ctx context.Context, req *spdkrpc.DiskGetRequest) (ret *spdkrpc.Disk, err error) {
	s.RLock()
	disk := s.diskMap[req.DiskName]
	spdkClient := s.spdkClient
	s.RUnlock()

	if disk == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find disk %v", req.DiskName)
	}

	return disk.DiskGet(spdkClient, req.DiskName, req.DiskPath, req.DiskDriver)
}

func (s *Server) DiskHealthGet(ctx context.Context, req *spdkrpc.DiskHealthGetRequest) (ret *spdkrpc.DiskHealthGetResponse, err error) {
	s.RLock()
	disk := s.diskMap[req.DiskName]
	spdkClient := s.spdkClient
	s.RUnlock()

	if disk == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "disk %q not found", req.DiskName)
	}

	diskHealth, err := disk.diskHealthGet(spdkClient, req.DiskName, req.DiskDriver)
	if err != nil {
		return nil, err
	}

	return &spdkrpc.DiskHealthGetResponse{
		ModelNumber:                             diskHealth.ModelNumber,
		SerialNumber:                            diskHealth.SerialNumber,
		FirmwareRevision:                        diskHealth.FirmwareRevision,
		Traddr:                                  diskHealth.Traddr,
		CriticalWarning:                         diskHealth.CriticalWarning,
		TemperatureCelsius:                      diskHealth.TemperatureCelsius,
		AvailableSparePercentage:                diskHealth.AvailableSparePercentage,
		AvailableSpareThresholdPercentage:       diskHealth.AvailableSpareThresholdPercentage,
		PercentageUsed:                          diskHealth.PercentageUsed,
		DataUnitsRead:                           diskHealth.DataUnitsRead,
		DataUnitsWritten:                        diskHealth.DataUnitsWritten,
		HostReadCommands:                        diskHealth.HostReadCommands,
		HostWriteCommands:                       diskHealth.HostWriteCommands,
		ControllerBusyTime:                      diskHealth.ControllerBusyTime,
		PowerCycles:                             diskHealth.PowerCycles,
		PowerOnHours:                            diskHealth.PowerOnHours,
		UnsafeShutdowns:                         diskHealth.UnsafeShutdowns,
		MediaErrors:                             diskHealth.MediaErrors,
		NumErrLogEntries:                        diskHealth.NumErrLogEntries,
		WarningTemperatureTimeMinutes:           diskHealth.WarningTemperatureTimeMinutes,
		CriticalCompositeTemperatureTimeMinutes: diskHealth.CriticalCompositeTemperatureTimeMinutes,
	}, nil
}
</file>

<file path="pkg/spdk/server_engine.go">
package spdk

import (
	"context"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"google.golang.org/protobuf/types/known/emptypb"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/types/pkg/generated/spdkrpc"

	"github.com/longhorn/longhorn-spdk-engine/pkg/api"
	"github.com/longhorn/longhorn-spdk-engine/pkg/types"
	"github.com/longhorn/longhorn-spdk-engine/pkg/util"
)

// EngineCreate creates an engine
func (s *Server) EngineCreate(ctx context.Context, req *spdkrpc.EngineCreateRequest) (ret *spdkrpc.Engine, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine name is required")
	}
	if req.VolumeName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine volume name is required")
	}
	if req.SpecSize == 0 {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine spec size is required")
	}

	s.Lock()

	e, ok := s.engineMap[req.Name]
	if ok {
		s.Unlock()
		return nil, grpcstatus.Errorf(grpccodes.AlreadyExists, "engine %v already exists", req.Name)
	}

	if e == nil {
		s.engineMap[req.Name] = NewEngine(req.Name, req.VolumeName, req.Frontend, req.SpecSize, s.updateChs[types.InstanceTypeEngine])
		e = s.engineMap[req.Name]
	}

	spdkClient := s.spdkClient
	s.Unlock()

	return e.Create(spdkClient, req.ReplicaAddressMap, req.PortCount, s.portAllocator, req.SalvageRequested)
}

// EngineDelete deletes an engine
func (s *Server) EngineDelete(ctx context.Context, req *spdkrpc.EngineDeleteRequest) (ret *emptypb.Empty, err error) {
	s.RLock()
	e := s.engineMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	defer func() {
		if err == nil {
			s.Lock()
			delete(s.engineMap, req.Name)
			s.Unlock()
		}
	}()

	if e != nil {
		if err := e.Delete(spdkClient, s.portAllocator); err != nil {
			return nil, err
		}
	}

	return &emptypb.Empty{}, nil
}

// EngineGet returns a specific engine
func (s *Server) EngineExpand(ctx context.Context, req *spdkrpc.EngineExpandRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine name is required")
	}

	s.RLock()
	e := s.engineMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if e == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v for expansion", req.Name)
	}

	if types.IsUblkFrontend(e.Frontend) {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "cannot expand ublk frontend engine %v", req.Name)
	}

	err = e.Expand(spdkClient, req.Size)
	if err != nil {
		return nil, toExpansionGRPCError(err, "failed to expand engine %v", req.Name)
	}

	return &emptypb.Empty{}, nil
}

// EngineExpandPrecheck checks if expansion is required for an engine. The engine spec size should be updated before precheck.
func (s *Server) EngineExpandPrecheck(ctx context.Context, req *spdkrpc.EngineExpandPrecheckRequest) (*spdkrpc.EngineExpandPrecheckResponse, error) {
	if req.Name == "" {
		return &spdkrpc.EngineExpandPrecheckResponse{
			ExpansionRequired: false,
		}, grpcstatus.Error(grpccodes.InvalidArgument, "engine name is required")
	}

	s.RLock()
	e := s.engineMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if e == nil {
		return &spdkrpc.EngineExpandPrecheckResponse{}, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v for expansion", req.Name)
	}

	requireExpansion, err := e.ExpandPrecheck(spdkClient, req.Size)
	if err != nil {
		return &spdkrpc.EngineExpandPrecheckResponse{
			ExpansionRequired: false,
		}, toExpansionGRPCError(err, "failed to precheck expand engine %v", req.Name)
	}

	return &spdkrpc.EngineExpandPrecheckResponse{
		ExpansionRequired: requireExpansion,
	}, nil
}

// EngineFrontendSwitchOver switches over the frontend of an engine to a new target address. The engine frontend should be in normal state before switch over.
func (s *Server) EngineFrontendSwitchOver(ctx context.Context, req *spdkrpc.EngineFrontendSwitchOverRequest) (ret *emptypb.Empty, err error) {
	if req == nil {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "request is required")
	}
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine frontend or engine name is required")
	}
	if req.TargetAddress == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "target address is required")
	}
	if targetIP, targetPort, splitErr := splitHostPort(req.TargetAddress); splitErr != nil || targetIP == "" || targetPort == 0 {
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "invalid target address %q", req.TargetAddress)
	}

	s.RLock()
	ef := s.engineFrontendMap[req.Name]
	if ef == nil {
		// Backward compatible lookup: allow name to be engine name if there is exactly one frontend.
		for _, frontend := range s.engineFrontendMap {
			if frontend.EngineName != req.Name {
				continue
			}
			if ef != nil {
				s.RUnlock()
				return nil, grpcstatus.Errorf(grpccodes.FailedPrecondition, "multiple engine frontends found for engine %s", req.Name)
			}
			ef = frontend
		}
	}
	spdkClient := s.spdkClient
	s.RUnlock()

	if ef == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine frontend or engine %v for target switchover", req.Name)
	}

	if err := ef.SwitchOverTarget(spdkClient, req.EngineName, req.TargetAddress); err != nil {
		return nil, toSwitchOverGRPCError(err, "failed to switch over target for %s", req.Name)
	}

	return &emptypb.Empty{}, nil
}

// EngineDeleteTarget deletes the target for an engine.
// TODO: The API is currently not implemented and will be removed in the future as target management will be handled by engine frontends instead of the engine itself.
func (s *Server) EngineDeleteTarget(ctx context.Context, req *spdkrpc.EngineDeleteTargetRequest) (ret *emptypb.Empty, err error) {
	return &emptypb.Empty{}, grpcstatus.Error(grpccodes.Unimplemented, "EngineDeleteTarget is not implemented yet and will be removed in the future")
}

// EngineGet returns a specific engine
func (s *Server) EngineGet(ctx context.Context, req *spdkrpc.EngineGetRequest) (ret *spdkrpc.Engine, err error) {
	s.RLock()
	e := s.engineMap[req.Name]
	s.RUnlock()

	if e == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v", req.Name)
	}

	return e.Get(), nil
}

// EngineList returns all engines
func (s *Server) EngineList(ctx context.Context, req *emptypb.Empty) (*spdkrpc.EngineListResponse, error) {
	engineMap := map[string]*Engine{}
	res := map[string]*spdkrpc.Engine{}

	s.RLock()
	for k, v := range s.engineMap {
		engineMap[k] = v
	}
	s.RUnlock()

	for engineName, e := range engineMap {
		res[engineName] = e.Get()
	}

	return &spdkrpc.EngineListResponse{Engines: res}, nil
}

// EngineWatch returns a stream of engine updates
func (s *Server) EngineWatch(req *emptypb.Empty, srv spdkrpc.SPDKService_EngineWatchServer) error {
	responseCh, err := s.Subscribe(types.InstanceTypeEngine)
	if err != nil {
		return err
	}

	defer func() {
		if err != nil {
			logrus.WithError(err).Error("SPDK service engine watch errored out")
		} else {
			logrus.Info("SPDK service engine watch ended successfully")
		}
	}()
	logrus.Info("Started new SPDK service engine update watch")

	done := false
	for {
		select {
		case <-s.ctx.Done():
			logrus.Info("spdk gRPC server: stopped engine watch due to the context done")
			done = true
		case <-responseCh:
			if err := srv.Send(&emptypb.Empty{}); err != nil {
				return err
			}
		}
		if done {
			break
		}
	}

	return nil
}

// EngineReplicaAdd handles the full replica-add lifecycle.
//
// When EngineFrontendName and EngineFrontendAddress are both provided, Engine
// calls back to the EngineFrontend for suspend/resume around the snapshot and
// finish steps. When both are omitted, ReplicaAdd runs without a frontendSuspendResumeWrapper,
// preserving the older direct-engine API behavior.
func (s *Server) EngineReplicaAdd(ctx context.Context, req *spdkrpc.EngineReplicaAddRequest) (ret *emptypb.Empty, err error) {
	if req.ReplicaName == "" || req.ReplicaAddress == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name and address are required")
	}

	efName := req.EngineFrontendName
	efAddress := req.EngineFrontendAddress
	if (efName == "") != (efAddress == "") {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine frontend name and address must be provided together")
	}

	s.RLock()
	e := s.engineMap[req.EngineName]
	spdkClient := s.spdkClient
	s.RUnlock()

	if e == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v for replica %s add", req.EngineName, req.ReplicaName)
	}

	var frontendSuspendResumeWrapper replicaAddFrontendSuspendResumeWrapper
	if efName != "" {
		log := logrus.WithFields(logrus.Fields{
			"engineName":     req.EngineName,
			"replicaName":    req.ReplicaName,
			"engineFrontend": efName,
		})
		frontendSuspendResumeWrapper = buildGRPCReplicaAddFrontendSuspendResumeWrapper(efName, efAddress, log)
	}

	if err := e.ReplicaAdd(spdkClient, req.ReplicaName, req.ReplicaAddress, req.FastSync, frontendSuspendResumeWrapper); err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to add replica %s to engine %s: %v", req.ReplicaName, req.EngineName, err)
	}
	return &emptypb.Empty{}, nil
}

// EngineReplicaList returns all replicas for an engine
func (s *Server) EngineReplicaList(ctx context.Context, req *spdkrpc.EngineReplicaListRequest) (ret *spdkrpc.EngineReplicaListResponse, err error) {
	s.RLock()
	e := s.engineMap[req.EngineName]
	spdkClient := s.spdkClient
	s.RUnlock()

	if e == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v for replica list", req.EngineName)
	}

	replicas, err := e.ReplicaList(spdkClient)
	if err != nil {
		return nil, err
	}

	ret = &spdkrpc.EngineReplicaListResponse{
		Replicas: map[string]*spdkrpc.Replica{},
	}

	for _, r := range replicas {
		ret.Replicas[r.Name] = api.ReplicaToProtoReplica(r)
	}

	return ret, nil
}

func (s *Server) EngineReplicaDelete(ctx context.Context, req *spdkrpc.EngineReplicaDeleteRequest) (ret *emptypb.Empty, err error) {
	s.RLock()
	e := s.engineMap[req.EngineName]
	spdkClient := s.spdkClient
	s.RUnlock()

	if e == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v for replica %s with address %s delete", req.EngineName, req.ReplicaName, req.ReplicaAddress)
	}

	if err := e.ReplicaDelete(spdkClient, req.ReplicaName, req.ReplicaAddress); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (s *Server) EngineSnapshotCreate(ctx context.Context, req *spdkrpc.SnapshotRequest) (ret *spdkrpc.SnapshotResponse, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine name are required")
	}

	s.RLock()
	e := s.engineMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if e == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v for snapshot creation", req.Name)
	}

	snapshotName, err := e.SnapshotCreate(spdkClient, req.SnapshotName)
	return &spdkrpc.SnapshotResponse{SnapshotName: snapshotName}, err
}

func (s *Server) EngineSnapshotDelete(ctx context.Context, req *spdkrpc.SnapshotRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" || req.SnapshotName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine name and snapshot name are required")
	}

	s.RLock()
	e := s.engineMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if e == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v for snapshot deletion", req.Name)
	}

	if err := e.SnapshotDelete(spdkClient, req.SnapshotName); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (s *Server) EngineSnapshotRevert(ctx context.Context, req *spdkrpc.SnapshotRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" || req.SnapshotName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine name and snapshot name are required")
	}

	s.RLock()
	e := s.engineMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if e == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v for snapshot revert", req.Name)
	}

	if err := e.SnapshotRevert(spdkClient, req.SnapshotName); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (s *Server) EngineSnapshotPurge(ctx context.Context, req *spdkrpc.SnapshotRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine name is required")
	}

	s.RLock()
	e := s.engineMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if e == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v for snapshot purge", req.Name)
	}

	if err := e.SnapshotPurge(spdkClient); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (s *Server) EngineSnapshotHash(ctx context.Context, req *spdkrpc.SnapshotHashRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" || req.SnapshotName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine name and snapshot name are required")
	}

	s.RLock()
	e := s.engineMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if e == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v for snapshot hash", req.Name)
	}

	if err := e.SnapshotHash(spdkClient, req.SnapshotName, req.Rehash); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (s *Server) EngineSnapshotHashStatus(ctx context.Context, req *spdkrpc.SnapshotHashStatusRequest) (ret *spdkrpc.EngineSnapshotHashStatusResponse, err error) {
	if req.Name == "" || req.SnapshotName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine name and snapshot name are required")
	}

	s.RLock()
	e := s.engineMap[req.Name]
	s.RUnlock()

	if e == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v for snapshot hash status", req.Name)
	}

	return e.SnapshotHashStatus(req.SnapshotName)
}

func (s *Server) EngineSnapshotClone(ctx context.Context, req *spdkrpc.EngineSnapshotCloneRequest) (ret *emptypb.Empty, err error) {
	if err := util.VerifyParams(
		util.Param{Name: "name", Value: req.Name},
		util.Param{Name: "snapshotName", Value: req.SnapshotName},
		util.Param{Name: "srcEngineName", Value: req.SrcEngineName},
		util.Param{Name: "srcEngineAddress", Value: req.SrcEngineAddress},
	); err != nil {
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "%v", err)
	}

	s.RLock()
	e := s.engineMap[req.Name]
	s.RUnlock()

	if e == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v for snapshot clone", req.Name)
	}

	if err := e.SnapshotClone(req.SnapshotName, req.SrcEngineName, req.SrcEngineAddress, req.CloneMode); err != nil {
		return nil, err
	}
	return &emptypb.Empty{}, nil
}

func (s *Server) EngineBackupCreate(ctx context.Context, req *spdkrpc.BackupCreateRequest) (ret *spdkrpc.BackupCreateResponse, err error) {
	s.RLock()
	e := s.engineMap[req.EngineName]
	s.RUnlock()

	if e == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v for backup creation", req.EngineName)
	}

	recv, err := e.BackupCreate(req.BackupName, req.VolumeName, req.EngineName, req.SnapshotName, req.BackingImageName, req.BackingImageChecksum,
		req.Labels, req.BackupTarget, req.Credential, req.ConcurrentLimit, req.CompressionMethod, req.StorageClassName, e.SpecSize)
	if err != nil {
		return nil, err
	}
	return &spdkrpc.BackupCreateResponse{
		Backup:         recv.BackupName,
		IsIncremental:  recv.IsIncremental,
		ReplicaAddress: recv.ReplicaAddress,
	}, nil
}

func (s *Server) EngineBackupStatus(ctx context.Context, req *spdkrpc.BackupStatusRequest) (*spdkrpc.BackupStatusResponse, error) {
	s.RLock()
	e := s.engineMap[req.EngineName]
	s.RUnlock()

	if e == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v for backup creation", req.EngineName)
	}

	return e.BackupStatus(req.Backup, req.ReplicaAddress)
}

func (s *Server) EngineBackupRestore(ctx context.Context, req *spdkrpc.EngineBackupRestoreRequest) (ret *spdkrpc.EngineBackupRestoreResponse, err error) {
	logrus.WithFields(logrus.Fields{
		"backup":       req.BackupUrl,
		"engine":       req.EngineName,
		"snapshotName": req.SnapshotName,
		"concurrent":   req.ConcurrentLimit,
	}).Info("Restoring backup")

	s.RLock()
	e := s.engineMap[req.EngineName]
	spdkClient := s.spdkClient
	s.RUnlock()

	if e == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v for restoring backup", req.EngineName)
	}

	return e.BackupRestore(spdkClient, req.BackupUrl, req.EngineName, req.SnapshotName, req.Credential, req.ConcurrentLimit)
}

func (s *Server) EngineRestoreStatus(ctx context.Context, req *spdkrpc.RestoreStatusRequest) (*spdkrpc.RestoreStatusResponse, error) {
	s.RLock()
	e := s.engineMap[req.EngineName]
	s.RUnlock()

	if e == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine %v for backup creation", req.EngineName)
	}

	resp, err := e.RestoreStatus()
	if err != nil {
		err = errors.Wrapf(err, "failed to get restore status for engine %v", req.EngineName)
		return nil, grpcstatus.Errorf(grpccodes.Internal, "%v", err)
	}
	return resp, nil
}
</file>

<file path="pkg/spdk/server_enginefrontend.go">
package spdk

import (
	"context"
	"net"
	"strconv"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"google.golang.org/protobuf/types/known/emptypb"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/types/pkg/generated/spdkrpc"

	commonnet "github.com/longhorn/go-common-libs/net"

	"github.com/longhorn/longhorn-spdk-engine/pkg/types"
)

// EngineFrontendSuspend suspends an engine frontend. The engine frontend can be resumed later. The engine frontend should be in normal state before suspension.
func (s *Server) EngineFrontendSuspend(ctx context.Context, req *spdkrpc.EngineFrontendSuspendRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine frontend name is required")
	}

	s.RLock()
	ef := s.engineFrontendMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if ef == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine frontend %v for suspension", req.Name)
	}

	err = ef.Suspend(spdkClient)
	if err != nil {
		return nil, toEngineFrontendLifecycleGRPCError(err, "failed to suspend engine frontend %v", req.Name)
	}

	return &emptypb.Empty{}, nil
}

// EngineFrontendResume resumes an engine frontend. The engine frontend should have been suspended before resumption.
func (s *Server) EngineFrontendResume(ctx context.Context, req *spdkrpc.EngineFrontendResumeRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine frontend name is required")
	}

	s.RLock()
	ef := s.engineFrontendMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if ef == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine frontend %v for resumption", req.Name)
	}

	err = ef.Resume(spdkClient)
	if err != nil {
		return nil, toEngineFrontendLifecycleGRPCError(err, "failed to resume engine frontend %v", req.Name)
	}

	return &emptypb.Empty{}, nil
}

// EngineFrontendReplicaAdd initiates a replica-add (rebuild) for the Engine
// that this EngineFrontend is connected to. It is the entry point called by
// the longhorn-manager control plane.
//
// # Design
//
// EngineFrontend (EF) and Engine may reside on different nodes, so this
// handler is a thin proxy:
//
//  1. Validate EF preconditions (not creating, switching over, expanding,
//     restoring, and in Running state).
//  2. Resolve the local pod IP so the Engine can call back to this EF for
//     suspend/resume during the finish step.
//  3. Create a gRPC client to the (potentially remote) Engine node.
//  4. Delegate to Server.EngineReplicaAdd on the Engine node, passing the
//     EF name and address in the request fields.
//
// # Engine-side flow (Server.EngineReplicaAdd → Engine.ReplicaAdd)
//
// Server.EngineReplicaAdd reads the EF fields from the request, then
// calls buildGRPCReplicaAddFrontendSuspendResumeWrapper to create a replicaAddFrontendSuspendResumeWrapper
// — a callback that will call back to this EF for suspend/resume during both
// the snapshot-creation step and the finish step. It then delegates to
// Engine.ReplicaAdd(frontendSuspendResumeWrapper).
//
// Engine.ReplicaAdd runs the synchronous part under the Engine lock:
//
//  0. Snapshot + setup (sync, under frontendSuspendResumeWrapper): suspend frontend via
//     frontendSuspendResumeWrapper → create rebuild snapshot, connect to src/dst replica
//     SPDK services, add dst replica head bdev to RAID, mark dst as ModeWO
//     → resume frontend.
//
// On return (with lock released), a deferred goroutine runs the remaining
// phases:
//
//  1. Shallow copy (async): iterate over snapshots and copy data from the
//     source replica to the destination replica via SPDK shallow copy.
//  2. Finish (async): orchestrates the finish step with the frontendSuspendResumeWrapper:
//     a. If shallow copy failed, mark dst replica as ModeERR, call the
//     real replicaAddFinish for SPDK resource cleanup (detach external
//     snapshot controller, stop expose).
//     b. If shallow copy succeeded, call frontendSuspendResumeWrapper(finish):
//     - frontendSuspendResumeWrapper (buildGRPCReplicaAddFrontendSuspendResumeWrapper) calls back to
//     EF via gRPC: Suspend → finish() → Resume
//     - finish() (replicaAddFinish) detaches the external snapshot
//     NVMe controller on the dst replica, stops the src replica from
//     exposing, and promotes the dst replica from ModeWO to ModeRW.
//     c. If finish fails and was never called (e.g. suspend failure in
//     frontendSuspendResumeWrapper), a cleanup call to the real replicaAddFinish runs
//     inside frontendSuspendResumeWrapper for SPDK resource cleanup.
//
// This call returns as soon as the synchronous part succeeds; the
// remaining phases run in the background goroutine. The caller can monitor
// progress by polling the Engine's ReplicaModeMap — the rebuilding replica
// appears as ModeWO until finished (ModeRW) or failed (ModeERR).
//
// # Error handling
//
// EF is stateless with respect to replica-add. Engine owns all state.
// If the Engine's async goroutine fails at any phase, it sets the replica
// to ModeERR and cleans up SPDK resources — without notifying EF.
//
// The frontendSuspendResumeWrapper (buildGRPCReplicaAddFrontendSuspendResumeWrapper) handles EF-unreachable
// scenarios gracefully:
//   - EF node down / pod deleted (GetServiceClient fails) → proceed with
//     the operation without suspension (no active I/O = suspension unnecessary).
//   - Suspend fails → proceed without suspension (data integrity is not
//     compromised; not proceeding would block the rebuild).
//   - Resume fails → log error but do not override the operation result;
//     longhorn-manager will detect the stuck-suspended EF and recover.
func (s *Server) EngineFrontendReplicaAdd(ctx context.Context, req *spdkrpc.EngineFrontendReplicaAddRequest) (ret *emptypb.Empty, err error) {
	if req.ReplicaName == "" || req.ReplicaAddress == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name and address are required")
	}

	s.RLock()
	ef, ok := s.engineFrontendMap[req.EngineFrontendName]
	if !ok {
		s.RUnlock()
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine frontend %v for replica %s add", req.EngineFrontendName, req.ReplicaName)
	}
	s.RUnlock()

	// Validate EF state and capture engine connection info under lock.
	ef.RLock()
	if ef.isCreating {
		ef.RUnlock()
		return nil, grpcstatus.Errorf(grpccodes.FailedPrecondition, "engine frontend %s is still creating", ef.Name)
	}
	if ef.isSwitchingOver {
		ef.RUnlock()
		return nil, grpcstatus.Errorf(grpccodes.FailedPrecondition, "engine frontend %s is switching over target", ef.Name)
	}
	if ef.isExpanding {
		ef.RUnlock()
		return nil, grpcstatus.Errorf(grpccodes.FailedPrecondition, "engine frontend %s expansion is in progress", ef.Name)
	}
	if ef.IsRestoring {
		ef.RUnlock()
		return nil, grpcstatus.Errorf(grpccodes.FailedPrecondition, "engine frontend %s restore is in progress", ef.Name)
	}
	if ef.State != types.InstanceStateRunning {
		ef.RUnlock()
		return nil, grpcstatus.Errorf(grpccodes.FailedPrecondition, "invalid state %v for engine frontend %s replica %s add", ef.State, ef.Name, req.ReplicaName)
	}
	engineIP := ef.EngineIP
	engineName := ef.EngineName
	ef.RUnlock()

	// Resolve the local node IP so Engine can call back to this EF
	// for suspend/resume during the finish step.
	localIP, err := commonnet.GetIPForPod()
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get local IP for engine frontend %s: %v", ef.Name, err)
	}
	efAddress := net.JoinHostPort(localIP, strconv.Itoa(types.SPDKServicePort))

	// Create a gRPC client to the (potentially remote) Engine node.
	engineAddress := net.JoinHostPort(engineIP, strconv.Itoa(types.SPDKServicePort))
	engineClient, err := GetServiceClient(engineAddress)
	if err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to get SPDK client for engine %s at %s: %v", engineName, engineAddress, err)
	}
	defer func() {
		if errClose := engineClient.Close(); errClose != nil {
			logrus.WithError(errClose).Warnf("Failed to close engine SPDK client for %s", engineName)
		}
	}()

	// Delegate to EngineReplicaAdd on the Engine node, passing EF info for callback.
	if err := engineClient.EngineReplicaAdd(engineName, req.ReplicaName, req.ReplicaAddress, req.FastSync, ef.Name, efAddress); err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to add replica %s on engine %s: %v", req.ReplicaName, engineName, err)
	}

	return &emptypb.Empty{}, nil
}

// EngineFrontendCreate creates a new engine frontend.
func (s *Server) EngineFrontendCreate(ctx context.Context, req *spdkrpc.EngineFrontendCreateRequest) (ret *spdkrpc.EngineFrontend, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine frontend name is required")
	}
	if req.VolumeName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "volume name is required")
	}
	if req.EngineName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine name is required")
	}
	if req.SpecSize == 0 {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "spec size is required")
	}

	if !types.IsFrontendSupported(req.Frontend) {
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "frontend %v is not supported", req.Frontend)
	}

	s.Lock()
	_, ok := s.engineFrontendMap[req.Name]
	if ok {
		s.Unlock()
		return nil, grpcstatus.Errorf(grpccodes.AlreadyExists, "engine frontend %v already exists", req.Name)
	}
	if existing := s.engineFrontendByVolumeName(req.VolumeName); existing != nil {
		s.Unlock()
		return nil, grpcstatus.Errorf(grpccodes.AlreadyExists, "engine frontend %v already exists for volume %v", existing.Name, req.VolumeName)
	}

	ef := NewEngineFrontend(req.Name, req.EngineName, req.VolumeName, req.Frontend, req.SpecSize,
		req.UblkQueueDepth, req.UblkNumberOfQueue, s.updateChs[types.InstanceTypeEngineFrontend])
	ef.metadataDir = s.metadataDir

	spdkClient := s.spdkClient
	s.Unlock()

	ret, createErr := ef.Create(spdkClient, req.TargetAddress)

	// Distinguish hard errors (validation / precondition) from runtime
	// failures (e.g. NVMe initiator can't connect).  Hard errors are
	// returned before Create mutates state, so the frontend must NOT be
	// registered.  Runtime failures leave the frontend in Error state;
	// we register it so callers can inspect and clean it up via Delete.
	if createErr != nil &&
		(errors.Is(createErr, ErrEngineFrontendCreateInvalidArgument) ||
			errors.Is(createErr, ErrEngineFrontendCreatePrecondition)) {
		return nil, toEngineFrontendCreateGRPCError(createErr, "failed to create engine frontend %v", req.Name)
	}

	s.Lock()
	// Re-check after Create() to guard against a concurrent create that
	// raced through the same window.
	duplicateName := false
	duplicateVolume := false
	var winner *EngineFrontend
	if existing, exists := s.engineFrontendMap[req.Name]; exists {
		duplicateName = true
		winner = existing
	} else if existing := s.engineFrontendByVolumeName(req.VolumeName); existing != nil {
		duplicateVolume = true
		winner = existing
	}
	if duplicateName || duplicateVolume {
		s.Unlock()
		// The race loser holds a fully-created frontend with real SPDK
		// resources (bdevs, NVMe controllers, etc.). Clean them up so
		// they don't leak.
		// Only clear metadataDir when the loser shares the same
		// volumeName as the winner — they use the same persistence
		// directory, so the loser's Delete() must not remove it.
		// When volumeNames differ, each has its own directory and the
		// loser should clean up its own record.
		if winner != nil && ef.VolumeName == winner.VolumeName {
			ef.metadataDir = ""
		}
		if deleteErr := ef.Delete(spdkClient); deleteErr != nil {
			logrus.WithError(deleteErr).Warnf("Failed to clean up race-loser engine frontend %v", req.Name)
		}
		// The loser's Create() may have overwritten the winner's
		// persistence record (both share the same volumeName key).
		// Re-persist the winner to restore correct on-disk state.
		// Hold the winner's read lock to prevent concurrent mutations
		// (e.g. Delete, switchover) from racing with the field reads
		// inside saveEngineFrontendRecord.
		if winner != nil && winner.metadataDir != "" && ef.VolumeName == winner.VolumeName {
			winner.RLock()
			if err := saveEngineFrontendRecord(winner.metadataDir, winner); err != nil {
				logrus.WithError(err).Warnf("Failed to re-persist winner engine frontend %v record after race", winner.Name)
			}
			winner.RUnlock()
		}
		if duplicateVolume {
			return nil, grpcstatus.Errorf(grpccodes.AlreadyExists, "engine frontend already exists for volume %v", req.VolumeName)
		}
		return nil, grpcstatus.Errorf(grpccodes.AlreadyExists, "engine frontend %v already exists", req.Name)
	}
	s.engineFrontendMap[req.Name] = ef
	s.Unlock()

	// Runtime failure: the frontend is registered in Error state so it
	// can be inspected and cleaned up via Delete.
	if createErr != nil {
		return ef.Get(), nil
	}

	return ret, nil
}

// EngineFrontendDelete deletes an engine frontend.
func (s *Server) EngineFrontendDelete(ctx context.Context, req *spdkrpc.EngineFrontendDeleteRequest) (ret *emptypb.Empty, err error) {
	s.RLock()
	ef := s.engineFrontendMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	defer func() {
		if err != nil {
			return
		}

		s.Lock()
		delete(s.engineFrontendMap, req.Name)
		s.Unlock()
	}()

	if ef == nil {
		return &emptypb.Empty{}, nil
	}

	if err := ef.Delete(spdkClient); err != nil {
		return nil, toEngineFrontendLifecycleGRPCError(err, "failed to delete engine frontend %v", req.Name)
	}

	return &emptypb.Empty{}, nil
}

// EngineFrontendGet returns a specific engine frontend
func (s *Server) EngineFrontendGet(ctx context.Context, req *spdkrpc.EngineFrontendGetRequest) (ret *spdkrpc.EngineFrontend, err error) {
	s.RLock()
	ef := s.engineFrontendMap[req.Name]
	s.RUnlock()

	if ef == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine frontend %v", req.Name)
	}

	return ef.Get(), nil
}

// EngineFrontendList lists all engine frontends.
func (s *Server) EngineFrontendList(ctx context.Context, req *emptypb.Empty) (*spdkrpc.EngineFrontendListResponse, error) {
	engineFrontendMap := map[string]*EngineFrontend{}
	res := map[string]*spdkrpc.EngineFrontend{}

	s.RLock()
	for k, v := range s.engineFrontendMap {
		engineFrontendMap[k] = v
	}
	s.RUnlock()

	for engineFrontendName, ef := range engineFrontendMap {
		res[engineFrontendName] = ef.Get()
	}

	return &spdkrpc.EngineFrontendListResponse{EngineFrontends: res}, nil
}

// EngineFrontendWatch watches engine frontends.
func (s *Server) EngineFrontendWatch(req *emptypb.Empty, srv spdkrpc.SPDKService_EngineFrontendWatchServer) error {
	responseCh, err := s.Subscribe(types.InstanceTypeEngineFrontend)
	if err != nil {
		return err
	}

	defer func() {
		if err != nil {
			logrus.WithError(err).Error("SPDK service engine frontend watch errored out")
		} else {
			logrus.Info("SPDK service engine frontend watch ended successfully")
		}
	}()
	logrus.Info("Started new SPDK service engine frontend update watch")

	done := false
	for {
		select {
		case <-s.ctx.Done():
			logrus.Info("spdk gRPC server: stopped engine target watch due to the context done")
			done = true
		case <-responseCh:
			if err := srv.Send(&emptypb.Empty{}); err != nil {
				return err
			}
		}
		if done {
			break
		}
	}

	return nil
}

func (s *Server) EngineFrontendExpand(ctx context.Context, req *spdkrpc.EngineFrontendExpandRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine frontend name is required")
	}

	s.RLock()
	ef := s.engineFrontendMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if ef == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine frontend %v", req.Name)
	}

	if types.IsUblkFrontend(ef.Frontend) {
		return nil, grpcstatus.Errorf(grpccodes.Unimplemented, "cannot expand ublk frontend engine %v", ef.Name)
	}

	err = ef.Expand(ctx, spdkClient, req.Size)
	if err != nil {
		return nil, toExpansionGRPCError(err, "failed to expand engine frontend %v", req.Name)
	}

	return &emptypb.Empty{}, nil
}

func (s *Server) EngineFrontendSnapshotCreate(ctx context.Context, req *spdkrpc.SnapshotRequest) (ret *spdkrpc.SnapshotResponse, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine frontend name is required")
	}

	s.RLock()
	ef := s.engineFrontendMap[req.Name]
	s.RUnlock()

	if ef == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine frontend %v for snapshot creation", req.Name)
	}

	snapshotName, err := ef.SnapshotCreate(req.SnapshotName)
	return &spdkrpc.SnapshotResponse{SnapshotName: snapshotName}, err
}

func (s *Server) EngineFrontendSnapshotDelete(ctx context.Context, req *spdkrpc.SnapshotRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" || req.SnapshotName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine frontend name and snapshot name are required")
	}

	s.RLock()
	ef := s.engineFrontendMap[req.Name]
	s.RUnlock()

	if ef == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine frontend %v for snapshot deletion", req.Name)
	}

	if err := ef.SnapshotDelete(req.SnapshotName); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (s *Server) EngineFrontendSnapshotRevert(ctx context.Context, req *spdkrpc.SnapshotRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" || req.SnapshotName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine frontend name and snapshot name are required")
	}

	s.RLock()
	ef := s.engineFrontendMap[req.Name]
	s.RUnlock()

	if ef == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine frontend %v for snapshot revert", req.Name)
	}

	if err := ef.SnapshotRevert(req.SnapshotName); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

func (s *Server) EngineFrontendSnapshotPurge(ctx context.Context, req *spdkrpc.SnapshotRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "engine frontend name is required")
	}

	s.RLock()
	ef := s.engineFrontendMap[req.Name]
	s.RUnlock()

	if ef == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine frontend %v for snapshot purge", req.Name)
	}

	if err := ef.SnapshotPurge(); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}
</file>

<file path="pkg/spdk/server_log.go">
package spdk

import (
	"context"

	"google.golang.org/protobuf/types/known/emptypb"

	"github.com/longhorn/types/pkg/generated/spdkrpc"
)

func (s *Server) LogSetLevel(ctx context.Context, req *spdkrpc.LogSetLevelRequest) (ret *emptypb.Empty, err error) {
	s.RLock()
	spdkClient := s.spdkClient
	s.RUnlock()

	err = svcLogSetLevel(spdkClient, req.Level)
	if err != nil {
		return nil, err
	}
	return &emptypb.Empty{}, nil
}

func (s *Server) LogSetFlags(ctx context.Context, req *spdkrpc.LogSetFlagsRequest) (ret *emptypb.Empty, err error) {
	s.RLock()
	spdkClient := s.spdkClient
	s.RUnlock()

	err = svcLogSetFlags(spdkClient, req.Flags)
	if err != nil {
		return nil, err
	}
	return &emptypb.Empty{}, nil
}

func (s *Server) LogGetLevel(ctx context.Context, req *emptypb.Empty) (ret *spdkrpc.LogGetLevelResponse, err error) {
	s.RLock()
	spdkClient := s.spdkClient
	s.RUnlock()

	level, err := svcLogGetLevel(spdkClient)
	if err != nil {
		return nil, err
	}

	return &spdkrpc.LogGetLevelResponse{
		Level: level,
	}, nil
}

func (s *Server) LogGetFlags(ctx context.Context, req *emptypb.Empty) (ret *spdkrpc.LogGetFlagsResponse, err error) {
	s.RLock()
	spdkClient := s.spdkClient
	s.RUnlock()

	flags, err := svcLogGetFlags(spdkClient)
	if err != nil {
		return nil, err
	}

	return &spdkrpc.LogGetFlagsResponse{
		Flags: flags,
	}, nil
}
</file>

<file path="pkg/spdk/server_replica.go">
package spdk

import (
	"context"
	"fmt"
	"net"
	"strconv"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"

	"google.golang.org/protobuf/types/known/emptypb"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/backupstore"
	"github.com/longhorn/types/pkg/generated/spdkrpc"

	butil "github.com/longhorn/backupstore/util"

	"github.com/longhorn/longhorn-spdk-engine/pkg/api"
	"github.com/longhorn/longhorn-spdk-engine/pkg/types"
	"github.com/longhorn/longhorn-spdk-engine/pkg/util"
)

func (s *Server) ReplicaCreate(ctx context.Context, req *spdkrpc.ReplicaCreateRequest) (ret *spdkrpc.Replica, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name is required")
	}
	if req.LvsName == "" && req.LvsUuid == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "either lvstore name or UUID is required")
	}

	r, err := s.newReplica(req)
	if err != nil {
		return nil, err
	}

	defer func() {
		// Always update the replica map
		s.Lock()
		s.replicaMap[req.Name] = r
		s.Unlock()
	}()

	s.RLock()
	spdkClient := s.spdkClient
	s.RUnlock()

	var backingImage *BackingImage
	if req.BackingImageName != "" {
		backingImage, err = s.getBackingImage(req.BackingImageName, req.LvsUuid)
		if err != nil {
			return nil, err
		}
	}

	return r.Create(spdkClient, req.PortCount, s.portAllocator, backingImage)
}

// ReplicaDelete deletes a replica
func (s *Server) ReplicaDelete(ctx context.Context, req *spdkrpc.ReplicaDeleteRequest) (ret *emptypb.Empty, err error) {
	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	defer func() {
		if err == nil && req.CleanupRequired {
			s.Lock()
			delete(s.replicaMap, req.Name)
			s.Unlock()
		}
	}()

	if r != nil {
		if err := r.Delete(spdkClient, req.CleanupRequired, s.portAllocator); err != nil {
			return nil, err
		}
	}

	return &emptypb.Empty{}, nil
}

// ReplicaGet returns a specific replica
func (s *Server) ReplicaGet(ctx context.Context, req *spdkrpc.ReplicaGetRequest) (ret *spdkrpc.Replica, err error) {
	s.RLock()
	r := s.replicaMap[req.Name]
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %v", req.Name)
	}

	return r.Get(), nil
}

// ReplicaExpand expands a replica
func (s *Server) ReplicaExpand(ctx context.Context, req *spdkrpc.ReplicaExpandRequest) (ret *emptypb.Empty, err error) {
	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %v", req.Name)
	}

	if err := r.Expand(spdkClient, req.Size); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

// ReplicaList returns all replicas
func (s *Server) ReplicaList(ctx context.Context, req *emptypb.Empty) (*spdkrpc.ReplicaListResponse, error) {
	replicaMap := map[string]*Replica{}
	res := map[string]*spdkrpc.Replica{}

	s.RLock()
	for k, v := range s.replicaMap {
		replicaMap[k] = v
	}
	s.RUnlock()

	for replicaName, r := range replicaMap {
		res[replicaName] = r.Get()
	}

	return &spdkrpc.ReplicaListResponse{Replicas: res}, nil
}

// ReplicaWatch returns a stream of replica updates
func (s *Server) ReplicaWatch(req *emptypb.Empty, srv spdkrpc.SPDKService_ReplicaWatchServer) error {
	responseCh, err := s.Subscribe(types.InstanceTypeReplica)
	if err != nil {
		return err
	}

	defer func() {
		if err != nil {
			logrus.WithError(err).Error("SPDK service replica watch errored out")
		} else {
			logrus.Info("SPDK service replica watch ended successfully")
		}
	}()
	logrus.Info("Started new SPDK service replica update watch")

	done := false
	for {
		select {
		case <-s.ctx.Done():
			logrus.Info("spdk gRPC server: stopped replica watch due to the context done")
			done = true
		case <-responseCh:
			if err := srv.Send(&emptypb.Empty{}); err != nil {
				return err
			}
		}
		if done {
			break
		}
	}

	return nil
}

// ReplicaSnapshotCreate creates a snapshot for a replica
func (s *Server) ReplicaSnapshotCreate(ctx context.Context, req *spdkrpc.SnapshotRequest) (ret *spdkrpc.Replica, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name is required")
	}
	if req.SnapshotName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "snapshot name is required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during snapshot create", req.Name)
	}

	opts := &api.SnapshotOptions{
		UserCreated: req.UserCreated,
		Timestamp:   req.SnapshotTimestamp,
	}

	return r.SnapshotCreate(spdkClient, req.SnapshotName, opts)
}

// ReplicaSnapshotDelete deletes a snapshot for a replica
func (s *Server) ReplicaSnapshotDelete(ctx context.Context, req *spdkrpc.SnapshotRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name is required")
	}
	if req.SnapshotName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "snapshot name is required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during snapshot delete", req.Name)
	}

	_, err = r.SnapshotDelete(spdkClient, req.SnapshotName)
	return &emptypb.Empty{}, err
}

// ReplicaSnapshotRevert reverts a snapshot for a replica
func (s *Server) ReplicaSnapshotRevert(ctx context.Context, req *spdkrpc.SnapshotRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" || req.SnapshotName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name and snapshot name are required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during snapshot revert", req.Name)
	}

	_, err = r.SnapshotRevert(spdkClient, req.SnapshotName)
	return &emptypb.Empty{}, err
}

// ReplicaSnapshotPurge purges all snapshots for a replica
func (s *Server) ReplicaSnapshotPurge(ctx context.Context, req *spdkrpc.SnapshotRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name is required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during snapshot purge", req.Name)
	}

	err = r.SnapshotPurge(spdkClient)
	return &emptypb.Empty{}, err
}

// ReplicaSnapshotHash hashes a snapshot for a replica
func (s *Server) ReplicaSnapshotHash(ctx context.Context, req *spdkrpc.SnapshotHashRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name is required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during snapshot hash", req.Name)
	}

	err = r.SnapshotHash(spdkClient, req.SnapshotName, req.Rehash)
	return &emptypb.Empty{}, err
}

// ReplicaSnapshotHashStatus returns the hash status of a snapshot for a replica
func (s *Server) ReplicaSnapshotHashStatus(ctx context.Context, req *spdkrpc.SnapshotHashStatusRequest) (ret *spdkrpc.ReplicaSnapshotHashStatusResponse, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name is required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s for snapshot hash status", req.Name)
	}

	state, checksum, errMsg, silentlyCorrupted, err := r.SnapshotHashStatus(req.SnapshotName)
	return &spdkrpc.ReplicaSnapshotHashStatusResponse{
		State:             state,
		Checksum:          checksum,
		Error:             errMsg,
		SilentlyCorrupted: silentlyCorrupted,
	}, err
}

// ReplicaSnapshotRangeHashGet returns the range hash of a snapshot for a replica
func (s *Server) ReplicaSnapshotRangeHashGet(ctx context.Context, req *spdkrpc.ReplicaSnapshotRangeHashGetRequest) (ret *spdkrpc.ReplicaSnapshotRangeHashGetResponse, err error) {
	if req.Name == "" || req.SnapshotName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name and snapshot name are required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s for snapshot range hash get", req.Name)
	}

	rangeHashMap, err := r.SnapshotRangeHashGet(spdkClient, req.SnapshotName, req.ClusterStartIndex, req.ClusterCount)
	return &spdkrpc.ReplicaSnapshotRangeHashGetResponse{
		RangeHashMap: rangeHashMap,
	}, err
}

// ReplicaSnapshotCloneDstStart starts a clone for a snapshot for a replica
func (s *Server) ReplicaSnapshotCloneDstStart(ctx context.Context, req *spdkrpc.ReplicaSnapshotCloneDstStartRequest) (ret *emptypb.Empty, err error) {
	if err := util.VerifyParams(
		util.Param{Name: "name", Value: req.Name},
		util.Param{Name: "snapshotName", Value: req.SnapshotName},
		util.Param{Name: "srcReplicaName", Value: req.SrcReplicaName},
		util.Param{Name: "srcReplicaAddress", Value: req.SrcReplicaAddress},
	); err != nil {
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "%v", err)
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during ReplicaSnapshotCloneDstStart", req.Name)
	}

	if err := r.SnapshotCloneDstStart(spdkClient, req.SnapshotName, req.SrcReplicaName, req.SrcReplicaAddress, req.CloneMode); err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to do SnapshotCloneDstStart during ReplicaSnapshotCloneDstStart")
	}
	return &emptypb.Empty{}, nil
}

// ReplicaSnapshotCloneDstStatusCheck checks the status of a clone for a snapshot for a replica
func (s *Server) ReplicaSnapshotCloneDstStatusCheck(ctx context.Context, req *spdkrpc.ReplicaSnapshotCloneDstStatusCheckRequest) (ret *spdkrpc.ReplicaSnapshotCloneDstStatusCheckResponse, err error) {
	if err := util.VerifyParams(
		util.Param{Name: "name", Value: req.Name},
	); err != nil {
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "%v", err)
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during ReplicaSnapshotCloneDstStatusCheck", req.Name)
	}

	return r.SnapshotCloneDstStatusCheck()
}

// ReplicaSnapshotCloneSrcStart starts a clone for a snapshot for a replica
func (s *Server) ReplicaSnapshotCloneSrcStart(ctx context.Context, req *spdkrpc.ReplicaSnapshotCloneSrcStartRequest) (ret *emptypb.Empty, err error) {
	if err := util.VerifyParams(
		util.Param{Name: "name", Value: req.Name},
		util.Param{Name: "snapshotName", Value: req.SnapshotName},
		util.Param{Name: "dstReplicaName", Value: req.DstReplicaName},
	); err != nil {
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "%v", err)
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during ReplicaSnapshotCloneSrcStart", req.Name)
	}

	if err := r.SnapshotCloneSrcStart(spdkClient, req.SnapshotName, req.DstReplicaName, req.DstCloningLvolAddress, req.CloneMode); err != nil {
		return nil, err
	}
	return &emptypb.Empty{}, nil
}

// ReplicaSnapshotCloneSrcStatusCheck checks the status of a clone for a snapshot for a replica
func (s *Server) ReplicaSnapshotCloneSrcStatusCheck(ctx context.Context, req *spdkrpc.ReplicaSnapshotCloneSrcStatusCheckRequest) (ret *spdkrpc.ReplicaSnapshotCloneSrcStatusCheckResponse, err error) {
	if err := util.VerifyParams(
		util.Param{Name: "name", Value: req.Name},
		util.Param{Name: "snapshotName", Value: req.SnapshotName},
		util.Param{Name: "dstReplicaName", Value: req.DstReplicaName},
	); err != nil {
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "%v", err)
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during ReplicaSnapshotCloneSrcStatusCheck", req.Name)
	}

	return r.SnapshotCloneSrcStatusCheck(spdkClient, req.SnapshotName, req.DstReplicaName)
}

// ReplicaSnapshotCloneSrcFinish finishes a clone for a snapshot for a replica
func (s *Server) ReplicaSnapshotCloneSrcFinish(ctx context.Context, req *spdkrpc.ReplicaSnapshotCloneSrcFinishRequest) (ret *emptypb.Empty, err error) {
	if err := util.VerifyParams(
		util.Param{Name: "name", Value: req.Name},
		util.Param{Name: "dstReplicaName", Value: req.DstReplicaName},
	); err != nil {
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "%v", err)
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during ReplicaSnapshotCloneSrcFinish", req.Name)
	}

	if err := r.SnapshotCloneSrcFinish(spdkClient, req.DstReplicaName); err != nil {
		return nil, err
	}
	return &emptypb.Empty{}, nil
}

// ReplicaRebuildingSrcStart starts a rebuilding for a replica
func (s *Server) ReplicaRebuildingSrcStart(ctx context.Context, req *spdkrpc.ReplicaRebuildingSrcStartRequest) (ret *spdkrpc.ReplicaRebuildingSrcStartResponse, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name is required")
	}
	if req.DstReplicaName == "" || req.DstReplicaAddress == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "dst replica name and address are required")
	}
	if req.ExposedSnapshotName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "src replica exposed snapshot name is required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during rebuilding src start", req.Name)
	}

	exposedSnapshotLvolAddress, err := r.RebuildingSrcStart(spdkClient, req.DstReplicaName, req.DstReplicaAddress, req.ExposedSnapshotName)
	if err != nil {
		return nil, err
	}
	return &spdkrpc.ReplicaRebuildingSrcStartResponse{ExposedSnapshotLvolAddress: exposedSnapshotLvolAddress}, nil
}

// ReplicaRebuildingSrcFinish finishes a rebuilding for a replica
func (s *Server) ReplicaRebuildingSrcFinish(ctx context.Context, req *spdkrpc.ReplicaRebuildingSrcFinishRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name is required")
	}
	if req.DstReplicaName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "dst replica name is required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during rebuilding src finish", req.Name)
	}

	if err = r.RebuildingSrcFinish(spdkClient, req.DstReplicaName); err != nil {
		return nil, err
	}
	return &emptypb.Empty{}, nil
}

// ReplicaRebuildingSrcShallowCopyStart starts a shallow copy for a rebuilding for a replica
func (s *Server) ReplicaRebuildingSrcShallowCopyStart(ctx context.Context, req *spdkrpc.ReplicaRebuildingSrcShallowCopyStartRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name is required")
	}
	if req.SnapshotName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica snapshot name is required")
	}
	if req.DstRebuildingLvolAddress == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "dst rebuilding lvol address is required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during rebuilding src snapshot %s shallow copy start", req.Name, req.SnapshotName)
	}

	if err := r.RebuildingSrcShallowCopyStart(spdkClient, req.SnapshotName, req.DstRebuildingLvolAddress); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

// ReplicaRebuildingSrcRangeShallowCopyStart starts a range shallow copy for a rebuilding for a replica
func (s *Server) ReplicaRebuildingSrcRangeShallowCopyStart(ctx context.Context, req *spdkrpc.ReplicaRebuildingSrcRangeShallowCopyStartRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name is required")
	}
	if req.SnapshotName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica snapshot name is required")
	}
	if req.DstRebuildingLvolAddress == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "dst rebuilding lvol address is required")
	}
	if len(req.MismatchingClusterList) < 1 {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "mismatching cluster list is required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during rebuilding src snapshot %s range shallow copy start", req.Name, req.SnapshotName)
	}

	if err := r.RebuildingSrcRangeShallowCopyStart(spdkClient, req.SnapshotName, req.DstRebuildingLvolAddress, req.MismatchingClusterList); err != nil {
		return nil, err
	}

	return &emptypb.Empty{}, nil
}

// ReplicaRebuildingSrcShallowCopyCheck checks the shallow copy for a rebuilding for a replica
func (s *Server) ReplicaRebuildingSrcShallowCopyCheck(ctx context.Context, req *spdkrpc.ReplicaRebuildingSrcShallowCopyCheckRequest) (ret *spdkrpc.ReplicaRebuildingSrcShallowCopyCheckResponse, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name is required")
	}
	if req.SnapshotName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica snapshot name is required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during rebuilding src snapshot %s shallow copy check", req.Name, req.SnapshotName)
	}

	return r.RebuildingSrcShallowCopyCheck(req.SnapshotName)
}

// ReplicaRebuildingDstStart starts a rebuilding for a replica
func (s *Server) ReplicaRebuildingDstStart(ctx context.Context, req *spdkrpc.ReplicaRebuildingDstStartRequest) (ret *spdkrpc.ReplicaRebuildingDstStartResponse, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name is required")
	}
	if req.SrcReplicaName == "" || req.SrcReplicaAddress == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "src replica name and address are required")
	}
	if req.ExternalSnapshotName == "" || req.ExternalSnapshotAddress == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "external external snapshot name and address are required")
	}
	if req.RebuildingSnapshotList == nil {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "rebuilding snapshot list is required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during rebuilding dst start", req.Name)
	}

	var rebuildingSnapshotList []*api.Lvol
	for _, snapshot := range req.RebuildingSnapshotList {
		rebuildingSnapshotList = append(rebuildingSnapshotList, api.ProtoLvolToLvol(snapshot))
	}
	address, err := r.RebuildingDstStart(spdkClient, req.SrcReplicaName, req.SrcReplicaAddress, req.ExternalSnapshotName, req.ExternalSnapshotAddress, rebuildingSnapshotList)
	if err != nil {
		return nil, err
	}
	return &spdkrpc.ReplicaRebuildingDstStartResponse{DstHeadLvolAddress: address}, nil
}

// ReplicaRebuildingDstFinish finishes a rebuilding for a replica
func (s *Server) ReplicaRebuildingDstFinish(ctx context.Context, req *spdkrpc.ReplicaRebuildingDstFinishRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name is required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during rebuilding dst finish", req.Name)
	}

	if err = r.RebuildingDstFinish(spdkClient); err != nil {
		return nil, err
	}
	return &emptypb.Empty{}, nil
}

// ReplicaRebuildingDstShallowCopyStart starts a shallow copy for a rebuilding for a replica
func (s *Server) ReplicaRebuildingDstShallowCopyStart(ctx context.Context, req *spdkrpc.ReplicaRebuildingDstShallowCopyStartRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name is required")
	}
	if req.SnapshotName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica snapshot name is required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during rebuilding dst snapshot %s shallow copy start", req.Name, req.SnapshotName)
	}

	if err = r.RebuildingDstShallowCopyStart(spdkClient, req.SnapshotName, req.FastSync); err != nil {
		return nil, err
	}
	return &emptypb.Empty{}, nil
}

// ReplicaRebuildingDstShallowCopyCheck checks the shallow copy for a rebuilding for a replica
func (s *Server) ReplicaRebuildingDstShallowCopyCheck(ctx context.Context, req *spdkrpc.ReplicaRebuildingDstShallowCopyCheckRequest) (ret *spdkrpc.ReplicaRebuildingDstShallowCopyCheckResponse, err error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name is required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during rebuilding dst snapshot shallow copy check", req.Name)
	}

	return r.RebuildingDstShallowCopyCheck(spdkClient)
}

// ReplicaRebuildingDstSnapshotCreate creates a snapshot for a rebuilding for a replica
func (s *Server) ReplicaRebuildingDstSnapshotCreate(ctx context.Context, req *spdkrpc.SnapshotRequest) (ret *emptypb.Empty, err error) {
	if req.Name == "" || req.SnapshotName == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name and snapshot name are required")
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s during rebuilding dst snapshot create", req.Name)
	}

	opts := &api.SnapshotOptions{
		UserCreated: req.UserCreated,
		Timestamp:   req.SnapshotTimestamp,
	}

	if err = r.RebuildingDstSnapshotCreate(spdkClient, req.SnapshotName, opts); err != nil {
		return nil, err
	}
	return &emptypb.Empty{}, nil
}

// ReplicaRebuildingDstSetQosLimit sets the QoS limit for a rebuilding for a replica
func (s *Server) ReplicaRebuildingDstSetQosLimit(
	ctx context.Context,
	req *spdkrpc.ReplicaRebuildingDstSetQosLimitRequest,
) (*emptypb.Empty, error) {
	if req.Name == "" {
		return nil, grpcstatus.Error(grpccodes.InvalidArgument, "replica name is required")
	}
	if req.QosLimitMbps < 0 {
		return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "QoS limit must not be negative, got %d", req.QosLimitMbps)
	}

	s.RLock()
	r := s.replicaMap[req.Name]
	spdkClient := s.spdkClient
	s.RUnlock()

	if r == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %s for QoS setting", req.Name)
	}

	if err := r.RebuildingDstSetQos(spdkClient, req.QosLimitMbps); err != nil {
		return nil, grpcstatus.Errorf(grpccodes.Internal, "failed to set QoS limit on replica %s: %v", req.Name, err)
	}

	return &emptypb.Empty{}, nil
}

func (s *Server) ReplicaBackupCreate(ctx context.Context, req *spdkrpc.BackupCreateRequest) (ret *spdkrpc.BackupCreateResponse, err error) {
	backupName := req.BackupName

	backupType, err := butil.CheckBackupType(req.BackupTarget)
	if err != nil {
		return nil, err
	}

	err = butil.SetupCredential(backupType, req.Credential)
	if err != nil {
		err = errors.Wrapf(err, "failed to setup credential of backup target %v for backup %v", req.BackupTarget, backupName)
		return nil, grpcstatus.Errorf(grpccodes.Internal, "%v", err)
	}

	var labelMap map[string]string
	if req.Labels != nil {
		labelMap, err = util.ParseLabels(req.Labels)
		if err != nil {
			err = errors.Wrapf(err, "failed to parse backup labels for backup %v", backupName)
			return nil, grpcstatus.Errorf(grpccodes.InvalidArgument, "%v", err)
		}
	}

	s.Lock()
	defer s.Unlock()

	if _, ok := s.backupMap[backupName]; ok {
		return nil, grpcstatus.Errorf(grpccodes.AlreadyExists, "backup %v already exists", backupName)
	}

	replica, ok := s.replicaMap[req.ReplicaName]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %v for volume %v backup creation", req.ReplicaName, req.VolumeName)
	}

	backup, err := NewBackup(s.spdkClient, backupName, req.VolumeName, req.SnapshotName, replica, s.portAllocator)
	if err != nil {
		err = errors.Wrapf(err, "failed to create backup instance %v for volume %v", backupName, req.VolumeName)
		return nil, grpcstatus.Errorf(grpccodes.Internal, "%v", err)
	}

	config := &backupstore.DeltaBackupConfig{
		BackupName:      backupName,
		ConcurrentLimit: req.ConcurrentLimit,
		Volume: &backupstore.Volume{
			Name:                 req.VolumeName,
			Size:                 req.Size,
			Labels:               labelMap,
			BackingImageName:     req.BackingImageName,
			BackingImageChecksum: req.BackingImageChecksum,
			CompressionMethod:    req.CompressionMethod,
			StorageClassName:     req.StorageClassName,
			CreatedTime:          util.Now(),
			DataEngine:           string(backupstore.DataEngineV2),
		},
		Snapshot: &backupstore.Snapshot{
			Name:        req.SnapshotName,
			CreatedTime: util.Now(),
		},
		DestURL:  req.BackupTarget,
		DeltaOps: backup,
		Labels:   labelMap,
	}

	s.backupMap[backupName] = backup
	if err := backup.BackupCreate(config); err != nil {
		delete(s.backupMap, backupName)
		err = errors.Wrapf(err, "failed to create backup %v for volume %v", backupName, req.VolumeName)
		return nil, grpcstatus.Errorf(grpccodes.Internal, "%v", err)
	}

	return &spdkrpc.BackupCreateResponse{
		Backup:        backup.Name,
		IsIncremental: backup.IsIncremental,
	}, nil
}

func (s *Server) ReplicaBackupStatus(ctx context.Context, req *spdkrpc.BackupStatusRequest) (ret *spdkrpc.BackupStatusResponse, err error) {
	s.RLock()
	defer s.RUnlock()

	backup, ok := s.backupMap[req.Backup]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find backup %v", req.Backup)
	}

	replicaAddress := ""
	if backup.replica != nil {
		replicaAddress = fmt.Sprintf("tcp://%s", backup.replica.GetAddress())
	}

	return &spdkrpc.BackupStatusResponse{
		Progress:       int32(backup.Progress),
		BackupUrl:      backup.BackupURL,
		Error:          backup.Error,
		SnapshotName:   backup.SnapshotName,
		State:          string(backup.State),
		ReplicaAddress: replicaAddress,
	}, nil
}

func (s *Server) ReplicaBackupRestore(ctx context.Context, req *spdkrpc.ReplicaBackupRestoreRequest) (ret *emptypb.Empty, err error) {
	s.RLock()
	replica := s.replicaMap[req.ReplicaName]
	spdkClient := s.spdkClient
	s.RUnlock()

	if replica == nil {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %v for restoring backup %v", req.ReplicaName, req.BackupUrl)
	}

	err = replica.BackupRestore(spdkClient, req.BackupUrl, req.SnapshotName, req.Credential, req.ConcurrentLimit)
	if err != nil {
		return nil, err
	}
	return &emptypb.Empty{}, nil
}

func (s *Server) ReplicaRestoreStatus(ctx context.Context, req *spdkrpc.ReplicaRestoreStatusRequest) (ret *spdkrpc.ReplicaRestoreStatusResponse, err error) {
	s.RLock()
	defer s.RUnlock()

	replica, ok := s.replicaMap[req.ReplicaName]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find replica %v", req.ReplicaName)
	}

	if replica.restore == nil {
		return &spdkrpc.ReplicaRestoreStatusResponse{
			ReplicaName: replica.Name,
			IsRestoring: false,
		}, nil
	}

	return &spdkrpc.ReplicaRestoreStatusResponse{
		ReplicaName:            replica.Name,
		ReplicaAddress:         net.JoinHostPort(replica.restore.ip, strconv.Itoa(int(replica.restore.port))),
		IsRestoring:            replica.isRestoring,
		LastRestored:           replica.restore.LastRestored,
		Progress:               int32(replica.restore.Progress),
		Error:                  replica.restore.Error,
		DestFileName:           replica.restore.LvolName,
		State:                  string(replica.restore.State),
		BackupUrl:              replica.restore.BackupURL,
		CurrentRestoringBackup: replica.restore.CurrentRestoringBackup,
	}, nil
}
</file>

<file path="pkg/spdk/server_verify_test.go">
package spdk

import (
	"context"
	"errors"
	"fmt"

	cockroacherrors "github.com/cockroachdb/errors"
	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/types/pkg/generated/spdkrpc"

	spdkjsonrpc "github.com/longhorn/go-spdk-helper/pkg/jsonrpc"
	spdktypes "github.com/longhorn/go-spdk-helper/pkg/spdk/types"

	lhtypes "github.com/longhorn/longhorn-spdk-engine/pkg/types"

	. "gopkg.in/check.v1"
)

func (s *TestSuite) TestNewReplicaExistingReplicaUpdatesMetadataIdempotently(c *C) {
	server := &Server{
		replicaMap: map[string]*Replica{
			"r1": NewReplica(context.Background(), "r1", "disk-a", "uuid-a", 1024, true, make(chan interface{}, 1)),
		},
	}

	replica, err := server.newReplica(&spdkrpc.ReplicaCreateRequest{
		Name:     "r1",
		LvsName:  "disk-b",
		LvsUuid:  "uuid-b",
		SpecSize: 2 << 20,
	})

	c.Assert(err, IsNil)
	c.Assert(replica, Equals, server.replicaMap["r1"])
	c.Assert(replica.SpecSize, Equals, uint64(2<<20))
	c.Assert(replica.LvsName, Equals, "disk-b")
	c.Assert(replica.LvsUUID, Equals, "uuid-b")
}

func (s *TestSuite) TestNewReplicaExistingReplicaAllowsMatchingMetadata(c *C) {
	server := &Server{
		replicaMap: map[string]*Replica{
			"r1": NewReplica(context.Background(), "r1", "disk-a", "uuid-a", 1024, true, make(chan interface{}, 1)),
		},
	}

	replica, err := server.newReplica(&spdkrpc.ReplicaCreateRequest{
		Name:     "r1",
		LvsName:  "disk-a",
		LvsUuid:  "uuid-a",
		SpecSize: 1 << 20,
	})

	c.Assert(err, IsNil)
	c.Assert(replica, Equals, server.replicaMap["r1"])
}

func (s *TestSuite) TestBuildBdevLvolMap(c *C) {
	fmt.Println("Testing buildBdevLvolMap with valid lvol, invalid lvol with extra alias, and non-lvol bdev")

	lvolValid := spdktypes.BdevInfo{
		BdevInfoBasic: spdktypes.BdevInfoBasic{
			Name:        "lvol-valid",
			Aliases:     []string{"lvs-a/replica-a"},
			ProductName: spdktypes.BdevProductNameLvol,
		},
		DriverSpecific: &spdktypes.BdevDriverSpecific{
			Lvol: &spdktypes.BdevDriverSpecificLvol{},
		},
	}
	lvolInvalidAlias := spdktypes.BdevInfo{
		BdevInfoBasic: spdktypes.BdevInfoBasic{
			Name:        "lvol-invalid-alias",
			Aliases:     []string{"lvs-a/replica-b", "extra"},
			ProductName: spdktypes.BdevProductNameLvol,
		},
		DriverSpecific: &spdktypes.BdevDriverSpecific{
			Lvol: &spdktypes.BdevDriverSpecificLvol{},
		},
	}
	raid := spdktypes.BdevInfo{
		BdevInfoBasic: spdktypes.BdevInfoBasic{
			Name:        "raid-a",
			ProductName: spdktypes.BdevProductNameRaid,
		},
		DriverSpecific: &spdktypes.BdevDriverSpecific{
			Raid: &spdktypes.BdevRaidInfo{},
		},
	}

	m := buildBdevLvolMap([]spdktypes.BdevInfo{lvolValid, lvolInvalidAlias, raid})
	c.Assert(len(m), Equals, 1)
	c.Assert(m["replica-a"], NotNil)
	c.Assert(m["replica-a"].Name, Equals, "lvol-valid")
}

func (s *TestSuite) TestBuildBdevLvolMapIgnoresInvalidDriverSpecific(c *C) {
	fmt.Println("Testing buildBdevLvolMap ignores invalid driver specific")

	lvolMissingDriverSpecific := spdktypes.BdevInfo{
		BdevInfoBasic: spdktypes.BdevInfoBasic{
			Name:        "lvol-invalid",
			Aliases:     []string{"lvs-a/replica-a"},
			ProductName: spdktypes.BdevProductNameLvol,
		},
		DriverSpecific: nil,
	}

	m := buildBdevLvolMap([]spdktypes.BdevInfo{lvolMissingDriverSpecific})
	c.Assert(len(m), Equals, 0)
}

func (s *TestSuite) TestBuildLvsUUIDNameMap(c *C) {
	fmt.Println("Testing buildLvsUUIDNameMap with valid lvs list")

	lvsList := []spdktypes.LvstoreInfo{
		{UUID: "uuid-a", Name: "disk-a"},
		{UUID: "uuid-b", Name: "disk-b"},
	}

	m := buildLvsUUIDNameMap(lvsList)
	c.Assert(len(m), Equals, 2)
	c.Assert(m["uuid-a"], Equals, "disk-a")
	c.Assert(m["uuid-b"], Equals, "disk-b")
}

func (s *TestSuite) TestHandleVerifyErrorBrokenPipe(c *C) {
	fmt.Println("Testing handleVerifyError with broken pipe error")

	replica := NewReplica(context.Background(), "r1", "disk-a", "uuid-a", 1024, true, make(chan interface{}, 1))
	engine := NewEngine("e1", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, make(chan interface{}, 1))
	engineFrontend := NewEngineFrontend("ef1", "e1", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1))

	replica.State = lhtypes.InstanceStateRunning
	engine.State = lhtypes.InstanceStateRunning
	engineFrontend.State = lhtypes.InstanceStateRunning

	state := &verifyState{
		replicaMapForSync: map[string]*Replica{
			"r1": replica,
		},
		engineMapForSync: map[string]*Engine{
			"e1": engine,
		},
		engineFrontendForSync: map[string]*EngineFrontend{
			"ef1": engineFrontend,
		},
	}

	brokenPipeErr := spdkjsonrpc.JSONClientError{
		ID:          1,
		Method:      "mock",
		ErrorDetail: errors.New("write: broken pipe"),
	}
	server := &Server{}
	server.handleVerifyError(brokenPipeErr, state)

	c.Assert(replica.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateError))
	c.Assert(engine.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateError))
	c.Assert(engineFrontend.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateError))
}

func (s *TestSuite) TestHandleVerifyErrorNonBrokenPipeNoStateChange(c *C) {
	fmt.Println("Testing handleVerifyError with non-broken pipe error does not change state")

	replica := NewReplica(context.Background(), "r1", "disk-a", "uuid-a", 1024, true, make(chan interface{}, 1))
	replica.State = lhtypes.InstanceStateRunning

	state := &verifyState{
		replicaMapForSync: map[string]*Replica{
			"r1": replica,
		},
		engineMapForSync:      map[string]*Engine{},
		engineFrontendForSync: map[string]*EngineFrontend{},
	}

	server := &Server{}
	server.handleVerifyError(errors.New("any other error"), state)

	c.Assert(replica.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateRunning))
}

func (s *TestSuite) TestHandleVerifyErrorNoopForNilError(c *C) {
	fmt.Println("Testing handleVerifyError with nil error does not change state")

	replica := NewReplica(context.Background(), "r1", "disk-a", "uuid-a", 1024, true, make(chan interface{}, 1))
	replica.State = lhtypes.InstanceStateRunning

	state := &verifyState{
		replicaMapForSync: map[string]*Replica{
			"r1": replica,
		},
		engineMapForSync:      map[string]*Engine{},
		engineFrontendForSync: map[string]*EngineFrontend{},
	}

	server := &Server{}
	server.handleVerifyError(nil, state)

	c.Assert(replica.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateRunning))
}

func (s *TestSuite) TestHandleVerifyErrorBrokenPipeKeepsStoppedAndError(c *C) {
	fmt.Println("Testing handleVerifyError with broken pipe error keeps stopped and error states")

	replicaStopped := NewReplica(context.Background(), "r-stopped", "disk-a", "uuid-a", 1024, true, make(chan interface{}, 1))
	replicaStopped.State = lhtypes.InstanceStateStopped

	engineErrored := NewEngine("e-err", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, make(chan interface{}, 1))
	engineErrored.State = lhtypes.InstanceStateError

	engineFrontendRunning := NewEngineFrontend("ef-run", "e-err", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1))
	engineFrontendRunning.State = lhtypes.InstanceStateRunning

	state := &verifyState{
		replicaMapForSync: map[string]*Replica{
			"r-stopped": replicaStopped,
		},
		engineMapForSync: map[string]*Engine{
			"e-err": engineErrored,
		},
		engineFrontendForSync: map[string]*EngineFrontend{
			"ef-run": engineFrontendRunning,
		},
	}

	brokenPipeErr := spdkjsonrpc.JSONClientError{
		ID:          1,
		Method:      "mock",
		ErrorDetail: errors.New("write: broken pipe"),
	}
	server := &Server{}
	server.handleVerifyError(brokenPipeErr, state)

	c.Assert(replicaStopped.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateStopped))
	c.Assert(engineErrored.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateError))
	c.Assert(engineFrontendRunning.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateError))
}

func (s *TestSuite) TestNewVerifyStateLockedCopiesMaps(c *C) {
	fmt.Println("Testing newVerifyState creates copies of maps while locked")

	server := &Server{
		replicaMap: map[string]*Replica{
			"r1": NewReplica(context.Background(), "r1", "disk-a", "uuid-a", 1024, true, make(chan interface{}, 1)),
		},
		engineMap: map[string]*Engine{
			"e1": NewEngine("e1", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, make(chan interface{}, 1)),
		},
		engineFrontendMap: map[string]*EngineFrontend{
			"ef1": NewEngineFrontend("ef1", "e1", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1)),
		},
		backingImageMap: map[string]*BackingImage{
			"bi1": NewBackingImage(context.Background(), "bi1", "uuid-bi1", "disk-uuid", 1024, "checksum", make(chan interface{}, 1)),
		},
		spdkClient: nil,
	}

	server.Lock()
	state := server.newVerifyState()
	server.Unlock()

	c.Assert(len(state.replicaMap), Equals, 1)
	c.Assert(len(state.replicaMapForSync), Equals, 1)
	c.Assert(len(state.engineMapForSync), Equals, 1)
	c.Assert(len(state.engineFrontendForSync), Equals, 1)
	c.Assert(len(state.backingImageMap), Equals, 1)
	c.Assert(len(state.backingImageForSync), Equals, 1)
	c.Assert(state.spdkClient, IsNil)

	_, ok := state.replicaMap["r1"]
	c.Assert(ok, Equals, true)
	_, ok = state.engineMapForSync["e1"]
	c.Assert(ok, Equals, true)
	_, ok = state.engineFrontendForSync["ef1"]
	c.Assert(ok, Equals, true)
	_, ok = state.backingImageMap["bi1"]
	c.Assert(ok, Equals, true)
}

func (s *TestSuite) TestSyncVerifiedObjectsWithEmptyState(c *C) {
	fmt.Println("Testing syncVerifiedObjects with empty state")

	server := &Server{}
	state := &verifyState{
		replicaMapForSync:     map[string]*Replica{},
		engineMapForSync:      map[string]*Engine{},
		engineFrontendForSync: map[string]*EngineFrontend{},
		backingImageForSync:   map[string]*BackingImage{},
		spdkClient:            nil,
	}

	err := server.syncVerifiedObjects(state)
	c.Assert(err, IsNil)
}

func (s *TestSuite) TestEngineFrontendCreateRegistersNewFrontend(c *C) {
	fmt.Println("Testing EngineFrontendCreate registers a new frontend in the map")

	srv := &Server{
		engineFrontendMap: map[string]*EngineFrontend{},
		updateChs: map[lhtypes.InstanceType]chan interface{}{
			lhtypes.InstanceTypeEngineFrontend: make(chan interface{}, 1),
		},
	}

	_, err := srv.EngineFrontendCreate(context.Background(), &spdkrpc.EngineFrontendCreateRequest{
		Name:       "ef-test",
		EngineName: "engine-a",
		VolumeName: "vol-a",
		Frontend:   lhtypes.FrontendSPDKTCPNvmf,
		SpecSize:   1024,
	})
	c.Assert(err, IsNil)

	srv.RLock()
	ef, ok := srv.engineFrontendMap["ef-test"]
	srv.RUnlock()

	c.Assert(ok, Equals, true)
	c.Assert(ef, NotNil)
	c.Assert(ef.Name, Equals, "ef-test")
	c.Assert(ef.EngineName, Equals, "engine-a")
	c.Assert(ef.VolumeName, Equals, "vol-a")
}

func (s *TestSuite) TestEngineFrontendCreateReturnsAlreadyExistsForDuplicate(c *C) {
	fmt.Println("Testing EngineFrontendCreate returns AlreadyExists for duplicate name")

	updateCh := make(chan interface{}, 1)

	existing := NewEngineFrontend("ef-dup", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPNvmf, 1024, 0, 0, updateCh)

	srv := &Server{
		engineFrontendMap: map[string]*EngineFrontend{
			"ef-dup": existing,
		},
		updateChs: map[lhtypes.InstanceType]chan interface{}{
			lhtypes.InstanceTypeEngineFrontend: updateCh,
		},
	}

	_, err := srv.EngineFrontendCreate(context.Background(), &spdkrpc.EngineFrontendCreateRequest{
		Name:       "ef-dup",
		EngineName: "engine-b",
		VolumeName: "vol-b",
		Frontend:   lhtypes.FrontendSPDKTCPNvmf,
		SpecSize:   2048,
	})
	c.Assert(err, NotNil)

	st, ok := grpcstatus.FromError(err)
	c.Assert(ok, Equals, true)
	c.Assert(st.Code(), Equals, grpccodes.AlreadyExists)

	// Original frontend should be untouched
	srv.RLock()
	ef := srv.engineFrontendMap["ef-dup"]
	srv.RUnlock()
	c.Assert(ef.EngineName, Equals, "engine-a")
}

func (s *TestSuite) TestEngineFrontendCreateDoesNotRegisterFailedFrontend(c *C) {
	fmt.Println("Testing EngineFrontendCreate does not leave a stale frontend after target address validation fails")

	srv := &Server{
		engineFrontendMap: map[string]*EngineFrontend{},
		updateChs: map[lhtypes.InstanceType]chan interface{}{
			lhtypes.InstanceTypeEngineFrontend: make(chan interface{}, 1),
		},
	}

	// "2001:db8::1:9502" is intentionally an un-bracketed IPv6 with
	// port, which net.SplitHostPort cannot parse. This triggers a hard
	// error from Create(), verifying that the failed frontend is NOT
	// registered in the map.
	_, err := srv.EngineFrontendCreate(context.Background(), &spdkrpc.EngineFrontendCreateRequest{
		Name:          "ef-test",
		EngineName:    "engine-a",
		VolumeName:    "vol-a",
		Frontend:      lhtypes.FrontendSPDKTCPNvmf,
		SpecSize:      1024,
		TargetAddress: "2001:db8::1:9502",
	})
	c.Assert(err, NotNil)
	st, ok := grpcstatus.FromError(err)
	c.Assert(ok, Equals, true)
	c.Assert(st.Code(), Equals, grpccodes.InvalidArgument)

	srv.RLock()
	_, exists := srv.engineFrontendMap["ef-test"]
	srv.RUnlock()
	c.Assert(exists, Equals, false)

	// Retry with empty TargetAddress. splitHostPort("") returns ("", 0, nil)
	// so Create() proceeds without initiator work and succeeds, proving
	// the name is no longer blocked by the earlier failure.
	_, err = srv.EngineFrontendCreate(context.Background(), &spdkrpc.EngineFrontendCreateRequest{
		Name:       "ef-test",
		EngineName: "engine-a",
		VolumeName: "vol-a",
		Frontend:   lhtypes.FrontendSPDKTCPNvmf,
		SpecSize:   1024,
	})
	c.Assert(err, IsNil)

	srv.RLock()
	ef, exists := srv.engineFrontendMap["ef-test"]
	srv.RUnlock()
	c.Assert(exists, Equals, true)
	c.Assert(ef, NotNil)
}

func (s *TestSuite) TestToEngineFrontendCreateGRPCErrorMapsKnownErrors(c *C) {
	fmt.Println("Testing toEngineFrontendCreateGRPCError maps known create failures to stable gRPC codes")

	testCases := []struct {
		name         string
		err          error
		expectedCode grpccodes.Code
	}{
		{
			name:         "invalid argument",
			err:          cockroacherrors.Wrap(ErrEngineFrontendCreateInvalidArgument, "bad target"),
			expectedCode: grpccodes.InvalidArgument,
		},
		{
			name:         "failed precondition",
			err:          cockroacherrors.Wrap(ErrEngineFrontendCreatePrecondition, "invalid state"),
			expectedCode: grpccodes.FailedPrecondition,
		},
		{
			name:         "existing grpc status preserved",
			err:          grpcstatus.Error(grpccodes.Unavailable, "transient"),
			expectedCode: grpccodes.Unavailable,
		},
	}

	for _, tc := range testCases {
		grpcErr := toEngineFrontendCreateGRPCError(tc.err, "failed to create engine frontend %v", "ef-test")
		st, ok := grpcstatus.FromError(grpcErr)
		c.Assert(ok, Equals, true, Commentf("case=%s", tc.name))
		c.Assert(st.Code(), Equals, tc.expectedCode, Commentf("case=%s", tc.name))
	}
}

func (s *TestSuite) TestEngineFrontendLifecycleRPCsMapKnownErrors(c *C) {
	fmt.Println("Testing EngineFrontend lifecycle RPCs map precondition and unimplemented errors consistently")

	newServer := func(ef *EngineFrontend) *Server {
		return &Server{
			engineFrontendMap: map[string]*EngineFrontend{
				"ef-test": ef,
			},
		}
	}

	suspendPrecondition := NewEngineFrontend("ef-test", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1))
	suspendPrecondition.State = lhtypes.InstanceStateRunning
	suspendPrecondition.isSwitchingOver = true

	_, err := newServer(suspendPrecondition).EngineFrontendSuspend(context.Background(), &spdkrpc.EngineFrontendSuspendRequest{Name: "ef-test"})
	st, ok := grpcstatus.FromError(err)
	c.Assert(ok, Equals, true)
	c.Assert(st.Code(), Equals, grpccodes.FailedPrecondition)

	resumeUnimplemented := NewEngineFrontend("ef-test", "engine-a", "vol-a", lhtypes.FrontendEmpty, 1024, 0, 0, make(chan interface{}, 1))
	resumeUnimplemented.State = lhtypes.InstanceStateSuspended

	_, err = newServer(resumeUnimplemented).EngineFrontendResume(context.Background(), &spdkrpc.EngineFrontendResumeRequest{Name: "ef-test"})
	st, ok = grpcstatus.FromError(err)
	c.Assert(ok, Equals, true)
	c.Assert(st.Code(), Equals, grpccodes.Unimplemented)

	deletePrecondition := NewEngineFrontend("ef-test", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1))
	deletePrecondition.isSwitchingOver = true

	_, err = newServer(deletePrecondition).EngineFrontendDelete(context.Background(), &spdkrpc.EngineFrontendDeleteRequest{Name: "ef-test"})
	st, ok = grpcstatus.FromError(err)
	c.Assert(ok, Equals, true)
	c.Assert(st.Code(), Equals, grpccodes.FailedPrecondition)

	// Delete while isCreating should also return FailedPrecondition
	deleteWhileCreating := NewEngineFrontend("ef-test", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1))
	deleteWhileCreating.isCreating = true

	_, err = newServer(deleteWhileCreating).EngineFrontendDelete(context.Background(), &spdkrpc.EngineFrontendDeleteRequest{Name: "ef-test"})
	st, ok = grpcstatus.FromError(err)
	c.Assert(ok, Equals, true)
	c.Assert(st.Code(), Equals, grpccodes.FailedPrecondition)
}
</file>

<file path="pkg/spdk/server.go">
package spdk

import (
	"context"
	"fmt"
	"sync"
	"sync/atomic"
	"time"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"

	"google.golang.org/protobuf/types/known/emptypb"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/go-spdk-helper/pkg/jsonrpc"
	"github.com/longhorn/types/pkg/generated/spdkrpc"

	commonbitmap "github.com/longhorn/go-common-libs/bitmap"
	spdkclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
	spdktypes "github.com/longhorn/go-spdk-helper/pkg/spdk/types"

	"github.com/longhorn/longhorn-spdk-engine/pkg/types"
	"github.com/longhorn/longhorn-spdk-engine/pkg/util"
	"github.com/longhorn/longhorn-spdk-engine/pkg/util/broadcaster"
)

const (
	MonitorInterval = 3 * time.Second
)

type Server struct {
	spdkrpc.UnimplementedSPDKServiceServer
	sync.RWMutex

	diskCreateLock sync.Mutex
	hotplugActive  atomic.Bool // use atomic.Bool to avoid data races across goroutines.

	ctx context.Context

	spdkClient    *spdkclient.Client
	portAllocator *commonbitmap.Bitmap

	diskMap           map[string]*Disk
	replicaMap        map[string]*Replica
	engineMap         map[string]*Engine
	engineFrontendMap map[string]*EngineFrontend

	backupMap map[string]*Backup

	// We store BackingImage in each lvstore
	backingImageMap map[string]*BackingImage

	broadcasters map[types.InstanceType]*broadcaster.Broadcaster
	broadcastChs map[types.InstanceType]chan interface{}
	updateChs    map[types.InstanceType]chan interface{}

	currentBdevIostat *spdktypes.BdevIostatResponse
	bdevMetricMap     map[string]*spdkrpc.Metrics

	// metadataDir is the base path for persisting engine frontend records
	// (e.g. /var/lib/longhorn). If empty, persistence is disabled.
	metadataDir string
}

func NewServer(ctx context.Context, portStart, portEnd int32) (*Server, error) {
	cli, err := spdkclient.NewClient(ctx)
	if err != nil {
		return nil, err
	}

	bitmap, err := commonbitmap.NewBitmap(portStart, portEnd)
	if err != nil {
		return nil, err
	}

	if _, err = cli.BdevNvmeSetOptions(
		replicaCtrlrLossTimeoutSec,
		replicaReconnectDelaySec,
		replicaFastIOFailTimeoutSec,
		replicaTransportAckTimeout,
		replicaKeepAliveTimeoutMs); err != nil {
		return nil, errors.Wrap(err, "failed to set NVMe options")
	}

	broadcasters := map[types.InstanceType]*broadcaster.Broadcaster{}
	broadcastChs := map[types.InstanceType]chan interface{}{}
	updateChs := map[types.InstanceType]chan interface{}{}
	for _, t := range []types.InstanceType{types.InstanceTypeReplica, types.InstanceTypeEngine, types.InstanceTypeEngineFrontend, types.InstanceTypeBackingImage} {
		broadcasters[t] = &broadcaster.Broadcaster{}
		broadcastChs[t] = make(chan interface{})
		updateChs[t] = make(chan interface{})
	}

	s := &Server{
		ctx: ctx,

		hotplugActive: atomic.Bool{},

		spdkClient:    cli,
		portAllocator: bitmap,

		diskMap: map[string]*Disk{},

		replicaMap:        map[string]*Replica{},
		engineMap:         map[string]*Engine{},
		engineFrontendMap: map[string]*EngineFrontend{},

		backupMap: map[string]*Backup{},

		backingImageMap: map[string]*BackingImage{},

		broadcasters: broadcasters,
		broadcastChs: broadcastChs,
		updateChs:    updateChs,

		metadataDir: types.MetadataDir,
	}
	s.hotplugActive.Store(true)

	if _, err := s.broadcasters[types.InstanceTypeReplica].Subscribe(ctx, s.replicaBroadcastConnector); err != nil {
		return nil, err
	}
	if _, err := s.broadcasters[types.InstanceTypeEngine].Subscribe(ctx, s.engineBroadcastConnector); err != nil {
		return nil, err
	}
	if _, err := s.broadcasters[types.InstanceTypeEngineFrontend].Subscribe(ctx, s.engineFrontendBroadcastConnector); err != nil {
		return nil, err
	}
	if _, err := s.broadcasters[types.InstanceTypeBackingImage].Subscribe(ctx, s.backingImageBroadcastConnector); err != nil {
		return nil, err
	}

	// Start broadcasting before recovery so that UpdateCh sends inside
	// RecoverFromHost do not block on the unbuffered channel.
	go s.broadcasting()

	s.recoverEngineFrontends()

	// TODO: There is no need to maintain the replica map in cache when we can use one SPDK JSON API call to fetch the Lvol tree/chain info
	go s.monitoring()

	return s, nil
}

func (s *Server) monitoring() {
	ticker := time.NewTicker(MonitorInterval)
	defer ticker.Stop()

	done := false
	for {
		select {
		case <-s.ctx.Done():
			logrus.Info("spdk gRPC server: stopped monitoring replicas due to the context done")
			done = true
		case <-ticker.C:
			err := s.verify()
			if err == nil {
				break
			}

			logrus.WithError(err).Errorf("spdk gRPC server: failed to verify and update replica cache, will retry later")

			if jsonrpc.IsJSONRPCRespErrorBrokenPipe(err) || jsonrpc.IsJSONRPCRespErrorInvalidCharacter(err) {
				err = s.tryEnsureSPDKTgtConnectionHealthy()
				if err != nil {
					logrus.WithError(err).Error("spdk gRPC server: failed to ensure spdk_tgt connection healthy")
				}
			}
		}
		if done {
			break
		}
	}
}

func (s *Server) tryEnsureSPDKTgtConnectionHealthy() error {
	running, err := util.IsSPDKTargetProcessRunning()
	if err != nil {
		return errors.Wrap(err, "failed to check spdk_tgt is running")
	}
	if !running {
		return errors.New("spdk_tgt is not running")
	}

	logrus.Info("spdk gRPC server: reconnecting to spdk_tgt")
	return s.clientReconnect()
}

func (s *Server) clientReconnect() error {
	s.Lock()
	defer func() {
		s.Unlock()
	}()

	oldClient := s.spdkClient

	client, err := spdkclient.NewClient(s.ctx)
	if err != nil {
		return errors.Wrap(err, "failed to create new SPDK client")
	}
	s.spdkClient = client

	// Try the best effort to close the old client after a new client is created
	err = oldClient.Close()
	if err != nil {
		logrus.WithError(err).Warn("Failed to close old SPDK client")
	}
	return nil
}

type verifyState struct {
	replicaMap            map[string]*Replica
	replicaMapForSync     map[string]*Replica
	engineMapForSync      map[string]*Engine
	engineFrontendForSync map[string]*EngineFrontend
	backingImageMap       map[string]*BackingImage
	backingImageForSync   map[string]*BackingImage
	spdkClient            *spdkclient.Client
}

func (s *Server) verify() (err error) {
	s.Lock()
	locked := true
	defer func() {
		if locked {
			s.Unlock()
		}
	}()

	state := s.newVerifyState()

	defer func() {
		s.handleVerifyError(err, state)
	}()

	s.trySelfHealHotplug()

	if err = s.rebuildCachedLvolObjects(state); err != nil {
		return err
	}

	if len(s.replicaMap) != len(state.replicaMap) {
		logrus.Infof("spdk gRPC server: replica map updated, map count is changed from %d to %d", len(s.replicaMap), len(state.replicaMap))
	}

	s.replicaMap = state.replicaMap
	s.backingImageMap = state.backingImageMap
	s.UpdateEngineMetrics()

	s.Unlock()
	locked = false

	return s.syncVerifiedObjects(state)
}

func (s *Server) newVerifyState() *verifyState {
	state := &verifyState{
		replicaMap:            map[string]*Replica{},
		replicaMapForSync:     map[string]*Replica{},
		engineMapForSync:      map[string]*Engine{},
		engineFrontendForSync: map[string]*EngineFrontend{},
		backingImageMap:       map[string]*BackingImage{},
		backingImageForSync:   map[string]*BackingImage{},
		spdkClient:            s.spdkClient,
	}

	for k, v := range s.replicaMap {
		state.replicaMap[k] = v
		state.replicaMapForSync[k] = v
	}
	for k, v := range s.engineMap {
		state.engineMapForSync[k] = v
	}
	for k, v := range s.engineFrontendMap {
		state.engineFrontendForSync[k] = v
	}
	for k, v := range s.backingImageMap {
		state.backingImageMap[k] = v
		state.backingImageForSync[k] = v
	}

	return state
}

func (s *Server) handleVerifyError(err error, state *verifyState) {
	if err == nil {
		return
	}
	if jsonrpc.IsJSONRPCRespErrorBrokenPipe(err) {
		logrus.WithError(err).Warn("spdk gRPC server: marking all non-stopped and non-error replicas and engines as error")
		for _, r := range state.replicaMapForSync {
			r.SetErrorState()
		}
		for _, e := range state.engineMapForSync {
			e.SetErrorState()
		}
		for _, ef := range state.engineFrontendForSync {
			ef.SetErrorState()
		}
	}
}

func (s *Server) trySelfHealHotplug() {
	// Self-heal: re-enable hotplug only if no disks are being created and the last enablement failed.
	isDiskCreating := false
	for _, disk := range s.diskMap {
		if disk.GetState() == DiskStateCreating {
			isDiskCreating = true
			break
		}
	}
	if !isDiskCreating && !s.hotplugActive.Load() {
		if success := setNvmeHotPlug(s.spdkClient, true); success {
			s.hotplugActive.Store(true)
		}
	}
}

func buildBdevLvolMap(bdevList []spdktypes.BdevInfo) map[string]*spdktypes.BdevInfo {
	bdevLvolMap := map[string]*spdktypes.BdevInfo{}
	for idx := range bdevList {
		bdev := &bdevList[idx]
		if spdktypes.GetBdevType(bdev) != spdktypes.BdevTypeLvol {
			continue
		}
		if len(bdev.Aliases) != 1 {
			continue
		}
		bdevLvolMap[spdktypes.GetLvolNameFromAlias(bdev.Aliases[0])] = bdev
	}
	return bdevLvolMap
}

func buildLvsUUIDNameMap(lvsList []spdktypes.LvstoreInfo) map[string]string {
	lvsUUIDNameMap := map[string]string{}
	for _, lvs := range lvsList {
		lvsUUIDNameMap[lvs.UUID] = lvs.Name
	}
	return lvsUUIDNameMap
}

func (s *Server) rebuildCachedLvolObjects(state *verifyState) error {
	bdevList, err := state.spdkClient.BdevGetBdevs("", 0)
	if err != nil {
		return err
	}
	bdevLvolMap := buildBdevLvolMap(bdevList)

	lvsList, err := state.spdkClient.BdevLvolGetLvstore("", "")
	if err != nil {
		return err
	}
	lvsUUIDNameMap := buildLvsUUIDNameMap(lvsList)

	// Detect if the lvol bdev is an uncached replica or backing image.
	for lvolName, bdevLvol := range bdevLvolMap {
		if bdevLvol.DriverSpecific.Lvol.Snapshot && !types.IsBackingImageSnapLvolName(lvolName) {
			continue
		}
		if types.IsBackingImageTempHead(lvolName) {
			if state.backingImageMap[types.GetBackingImageSnapLvolNameFromTempHeadLvolName(lvolName)] == nil {
				lvsUUID := bdevLvol.DriverSpecific.Lvol.LvolStoreUUID
				logrus.Infof("Found one backing image temp head lvol %v while there is no backing image record in the server", lvolName)
				if err := cleanupOrphanBackingImageTempHead(state.spdkClient, lvsUUIDNameMap[lvsUUID], lvolName); err != nil {
					logrus.WithError(err).Warnf("Failed to clean up orphan backing image temp head")
				}
			}
			continue
		}
		if state.replicaMap[lvolName] != nil {
			continue
		}
		if state.backingImageMap[lvolName] != nil {
			continue
		}
		if IsRebuildingLvol(lvolName) {
			if state.replicaMap[GetReplicaNameFromRebuildingLvolName(lvolName)] != nil {
				continue
			}
		}
		if IsCloningLvol(lvolName) {
			if state.replicaMap[GetReplicaNameFromCloningLvolName(lvolName)] != nil {
				continue
			}
		}
		if types.IsBackingImageSnapLvolName(lvolName) {
			lvsUUID := bdevLvol.DriverSpecific.Lvol.LvolStoreUUID
			backingImageName, _, err := ExtractBackingImageAndDiskUUID(lvolName)
			if err != nil {
				logrus.WithError(err).Warnf("failed to extract backing image name and disk UUID from lvol name %v", lvolName)
				continue
			}
			size := bdevLvol.NumBlocks * uint64(bdevLvol.BlockSize)
			alias := bdevLvol.Aliases[0]
			expectedChecksum, err := GetSnapXattr(state.spdkClient, alias, types.LonghornBackingImageSnapshotAttrChecksum)
			if err != nil {
				logrus.WithError(err).Warnf("failed to retrieve checksum attribute for backing image snapshot %v", alias)
				continue
			}
			backingImageUUID, err := GetSnapXattr(state.spdkClient, alias, types.LonghornBackingImageSnapshotAttrUUID)
			if err != nil {
				logrus.WithError(err).Warnf("failed to retrieve backing image UUID attribute for snapshot %v", alias)
				continue
			}
			backingImage := NewBackingImage(s.ctx, backingImageName, backingImageUUID, lvsUUID, size, expectedChecksum, s.updateChs[types.InstanceTypeBackingImage])
			backingImage.Alias = alias
			backingImage.State = types.BackingImageStatePending
			state.backingImageForSync[lvolName] = backingImage
			state.backingImageMap[lvolName] = backingImage
		} else if IsProbablyReplicaName(lvolName) {
			lvsUUID := bdevLvol.DriverSpecific.Lvol.LvolStoreUUID
			specSize := bdevLvol.NumBlocks * uint64(bdevLvol.BlockSize)
			actualSize := bdevLvol.DriverSpecific.Lvol.NumAllocatedClusters * uint64(defaultClusterSize)
			state.replicaMap[lvolName] = NewReplica(s.ctx, lvolName, lvsUUIDNameMap[lvsUUID], lvsUUID, specSize, true, s.updateChs[types.InstanceTypeReplica])
			state.replicaMapForSync[lvolName] = state.replicaMap[lvolName]
			logrus.Infof("Detected one possible existing replica %s(%s) with disk %s(%s), spec size %d, actual size %d", bdevLvol.Aliases[0], bdevLvol.UUID, lvsUUIDNameMap[lvsUUID], lvsUUID, specSize, actualSize)
		}
	}

	// Remove replicas from the cache if their lvol bdevs are gone.
	for replicaName, r := range state.replicaMap {
		// Try the best to avoid eliminating broken replicas or rebuilding replicas
		if bdevLvolMap[r.Name] == nil {
			if r.IsRebuilding() {
				continue
			}
			noReplicaLvol := true
			for lvolName := range bdevLvolMap {
				if IsReplicaLvol(r.Name, lvolName) {
					noReplicaLvol = false
					break
				}
			}
			if noReplicaLvol {
				delete(state.replicaMap, replicaName)
				delete(state.replicaMapForSync, replicaName)
			}
		}
	}

	return nil
}

func (s *Server) syncVerifiedObjects(state *verifyState) error {
	for _, r := range state.replicaMapForSync {
		if err := r.Sync(state.spdkClient); err != nil && jsonrpc.IsJSONRPCRespErrorBrokenPipe(err) {
			return err
		}
	}

	for _, e := range state.engineMapForSync {
		if err := e.ValidateAndUpdate(state.spdkClient); err != nil && jsonrpc.IsJSONRPCRespErrorBrokenPipe(err) {
			return err
		}
	}

	for _, ef := range state.engineFrontendForSync {
		if err := ef.ValidateAndUpdate(state.spdkClient); err != nil && jsonrpc.IsJSONRPCRespErrorBrokenPipe(err) {
			return err
		}
	}

	for _, bi := range state.backingImageForSync {
		if err := bi.ValidateAndUpdate(state.spdkClient); err != nil {
			if jsonrpc.IsJSONRPCRespErrorBrokenPipe(err) {
				return err
			}
			continue
		}
	}

	// TODO: send update signals if there is a Replica/Replica change
	return nil
}

func (s *Server) broadcasting() {
	for {
		select {
		case <-s.ctx.Done():
			logrus.Info("spdk gRPC server: stopped broadcasting instances due to the context done")
			// Keep draining updateChs so that senders on unbuffered channels
			// do not block forever after broadcasting stops forwarding.
			// Other goroutines will eventually observe ctx.Done() and stop sending.
			for {
				select {
				case <-s.updateChs[types.InstanceTypeReplica]:
				case <-s.updateChs[types.InstanceTypeEngine]:
				case <-s.updateChs[types.InstanceTypeEngineFrontend]:
				case <-s.updateChs[types.InstanceTypeBackingImage]:
				}
			}
		case <-s.updateChs[types.InstanceTypeReplica]:
			s.broadcastChs[types.InstanceTypeReplica] <- nil
		case <-s.updateChs[types.InstanceTypeEngine]:
			s.broadcastChs[types.InstanceTypeEngine] <- nil
		case <-s.updateChs[types.InstanceTypeEngineFrontend]:
			s.broadcastChs[types.InstanceTypeEngineFrontend] <- nil
		case <-s.updateChs[types.InstanceTypeBackingImage]:
			s.broadcastChs[types.InstanceTypeBackingImage] <- nil
		}
	}
}

func (s *Server) Subscribe(instanceType types.InstanceType) (<-chan interface{}, error) {
	switch instanceType {
	case types.InstanceTypeEngine:
		return s.broadcasters[types.InstanceTypeEngine].Subscribe(context.TODO(), s.engineBroadcastConnector)
	case types.InstanceTypeEngineFrontend:
		return s.broadcasters[types.InstanceTypeEngineFrontend].Subscribe(context.TODO(), s.engineFrontendBroadcastConnector)
	case types.InstanceTypeReplica:
		return s.broadcasters[types.InstanceTypeReplica].Subscribe(context.TODO(), s.replicaBroadcastConnector)
	case types.InstanceTypeBackingImage:
		return s.broadcasters[types.InstanceTypeBackingImage].Subscribe(context.TODO(), s.backingImageBroadcastConnector)
	}
	return nil, fmt.Errorf("invalid instance type %v for subscription", instanceType)
}

func (s *Server) replicaBroadcastConnector() (chan interface{}, error) {
	return s.broadcastChs[types.InstanceTypeReplica], nil
}

func (s *Server) engineBroadcastConnector() (chan interface{}, error) {
	return s.broadcastChs[types.InstanceTypeEngine], nil
}

func (s *Server) engineFrontendBroadcastConnector() (chan interface{}, error) {
	return s.broadcastChs[types.InstanceTypeEngineFrontend], nil
}

func (s *Server) backingImageBroadcastConnector() (chan interface{}, error) {
	return s.broadcastChs[types.InstanceTypeBackingImage], nil
}

func (s *Server) isLvsExist(lvsUUID, lvsName string) (bool, error) {
	if lvsUUID == "" && lvsName == "" {
		return false, fmt.Errorf("either lvstore UUID or name must be provided")
	}

	name := ""
	uuid := ""

	if lvsUUID != "" {
		uuid = lvsUUID
	} else {
		name = lvsName
	}

	lvsList, err := s.spdkClient.BdevLvolGetLvstore(name, uuid)
	if err != nil {
		return false, err
	}

	if len(lvsList) == 0 {
		return false, fmt.Errorf("found zero lvstore with name %q and UUID %q", lvsName, lvsUUID)
	}

	return true, nil
}

func (s *Server) newReplica(req *spdkrpc.ReplicaCreateRequest) (*Replica, error) {
	s.Lock()
	defer func() {
		s.Unlock()
	}()

	r, ok := s.replicaMap[req.Name]
	if ok {
		r.Lock()
		if req.SpecSize != 0 {
			r.SpecSize = req.SpecSize
		}
		if req.LvsName != "" {
			r.LvsName = req.LvsName
		}
		if req.LvsUuid != "" {
			r.LvsUUID = req.LvsUuid
		}
		r.Unlock()
		return r, nil
	}

	exists, err := s.isLvsExist(req.LvsUuid, req.LvsName)
	if err != nil {
		return nil, errors.Wrapf(err, "failed to check lvstore %v(%v) existence for replica %v creation", req.LvsName, req.LvsUuid, req.Name)
	}
	if !exists {
		return nil, fmt.Errorf("lvstore %v(%v) does not exist for replica %v creation", req.LvsName, req.LvsUuid, req.Name)
	}
	return NewReplica(s.ctx, req.Name, req.LvsName, req.LvsUuid, req.SpecSize, true, s.updateChs[types.InstanceTypeReplica]), nil
}

func (s *Server) getBackingImage(backingImageName, lvsUUID string) (backingImage *BackingImage, err error) {
	backingImageSnapLvolName := GetBackingImageSnapLvolName(backingImageName, lvsUUID)

	s.RLock()
	backingImage = s.backingImageMap[backingImageSnapLvolName]
	s.RUnlock()

	if backingImage == nil {
		return nil, grpcstatus.Error(grpccodes.NotFound, "failed to find the backing image in the spdk server")
	}

	return backingImage, nil
}

func toSwitchOverGRPCError(err error, format string, args ...interface{}) error {
	code := grpccodes.Internal

	// Preserve downstream gRPC code when available.
	if statusErr, ok := grpcstatus.FromError(errors.UnwrapAll(err)); ok {
		code = statusErr.Code()
	} else {
		switch {
		case errors.Is(err, ErrSwitchOverTargetInvalidInput):
			code = grpccodes.InvalidArgument
		case errors.Is(err, ErrSwitchOverTargetPrecondition):
			code = grpccodes.FailedPrecondition
		case errors.Is(err, ErrSwitchOverTargetEngineNotFound):
			code = grpccodes.NotFound
		case errors.Is(err, context.DeadlineExceeded):
			code = grpccodes.DeadlineExceeded
		case errors.Is(err, context.Canceled):
			code = grpccodes.Canceled
		}
	}

	return grpcstatus.Error(code, errors.Wrapf(err, format, args...).Error())
}

// buildGRPCReplicaAddFrontendSuspendResumeWrapper builds a replicaAddFrontendSuspendResumeWrapper that
// calls back to the EngineFrontend on a (potentially remote) node via gRPC
// for suspend/resume around the work step.
//
// If the EngineFrontend is unreachable (node down, pod deleted, etc.), the
// wrapper proceeds with work() without suspension. This is safe because an
// unreachable frontend means there is no active I/O to quiesce, and not
// running the work would leak SPDK resources (detach controller, stop expose).
func buildGRPCReplicaAddFrontendSuspendResumeWrapper(efName, efAddress string, log *logrus.Entry) replicaAddFrontendSuspendResumeWrapper {
	return func(work func() error) error {
		efClient, err := GetServiceClient(efAddress)
		if err != nil {
			// Cannot connect to the EF node at all — proceed without suspension.
			log.WithError(err).Warnf("Engine frontend %s at %s is unreachable, proceeding without suspension", efName, efAddress)
			return work()
		}
		defer func() {
			if errClose := efClient.Close(); errClose != nil {
				log.WithError(errClose).Warnf("Failed to close engine frontend SPDK client for %s", efName)
			}
		}()

		// Suspend the frontend before running the work.
		// If suspend fails for any reason (EF deleted, node down, unimplemented
		// frontend type), proceed without suspension rather than aborting. The
		// data has already been copied; not running the work is worse than a
		// brief I/O disruption.
		suspended := false
		if err := efClient.EngineFrontendSuspend(efName); err != nil {
			log.WithError(err).Warnf("Failed to suspend engine frontend %s before replica add work, proceeding without suspension", efName)
		} else {
			suspended = true
		}

		workErr := work()

		// Resume the frontend after the work.
		// If resume fails (EF disappeared during work, or internal error),
		// log a warning but do not override workErr — the replica-add result
		// is determined by work(), not by resume. longhorn-manager will
		// detect the stuck-suspended EF and handle recovery.
		if suspended {
			if resumeErr := efClient.EngineFrontendResume(efName); resumeErr != nil {
				log.WithError(resumeErr).Errorf("Failed to resume engine frontend %s after replica add work (work succeeded: %v)", efName, workErr == nil)
			}
		}

		return workErr
	}
}

func (s *Server) VersionDetailGet(context.Context, *emptypb.Empty) (*spdkrpc.VersionDetailGetReply, error) {
	// TODO: Implement this
	return &spdkrpc.VersionDetailGetReply{
		Version: &spdkrpc.VersionOutput{},
	}, nil
}

func (s *Server) newBackingImage(req *spdkrpc.BackingImageCreateRequest) (*BackingImage, error) {
	s.Lock()
	defer s.Unlock()

	// The backing image key is in this form "bi-%s-disk-%s" to distinguish different disks.
	backingImageSnapLvolName := GetBackingImageSnapLvolName(req.Name, req.LvsUuid)
	if _, ok := s.backingImageMap[backingImageSnapLvolName]; !ok {
		exists, err := s.isLvsExist(req.LvsUuid, "")
		if err != nil || !exists {
			return nil, err
		}
		s.backingImageMap[backingImageSnapLvolName] = NewBackingImage(s.ctx, req.Name, req.BackingImageUuid, req.LvsUuid, req.Size, req.Checksum, s.updateChs[types.InstanceTypeBackingImage])
	}

	return s.backingImageMap[backingImageSnapLvolName], nil
}

func (s *Server) UpdateEngineMetrics() {
	previousBdevIostat := s.currentBdevIostat
	bdevIostat, err := s.spdkClient.BdevGetIostat("", false)
	if err != nil {
		logrus.WithError(err).Error("failed to get bdev iostat")
		return
	}
	s.currentBdevIostat = bdevIostat
	// If this is the first execution, there is no previous data, so exit
	if previousBdevIostat == nil {
		return
	}
	// Calculate the elapsed time in ticks
	tickElapsed := s.currentBdevIostat.Ticks - previousBdevIostat.Ticks
	if tickElapsed == 0 {
		return
	}
	// Convert ticks to seconds
	elapsedSeconds := float64(tickElapsed) / float64(s.currentBdevIostat.TickRate)

	// Convert previous Bdev data into a map for quick lookup
	prevBdevMap := make(map[string]spdktypes.BdevStats)
	for _, prevBdev := range previousBdevIostat.Bdevs {
		prevBdevMap[prevBdev.Name] = prevBdev
	}

	// Initialize a new BdevMetricMap
	s.bdevMetricMap = make(map[string]*spdkrpc.Metrics)
	for _, bdev := range s.currentBdevIostat.Bdevs {
		prevBdev, exists := prevBdevMap[bdev.Name]
		if !exists {
			continue
		}

		// Calculate differences in operations and data
		readOpsDiff := bdev.NumReadOps - prevBdev.NumReadOps
		writeOpsDiff := bdev.NumWriteOps - prevBdev.NumWriteOps
		bytesReadDiff := bdev.BytesRead - prevBdev.BytesRead
		bytesWrittenDiff := bdev.BytesWritten - prevBdev.BytesWritten
		readLatencyDiff := bdev.ReadLatencyTicks - prevBdev.ReadLatencyTicks
		writeLatencyDiff := bdev.WriteLatencyTicks - prevBdev.WriteLatencyTicks

		// Convert latency from ticks to nanoseconds
		readLatency := calculateLatencyInNs(readLatencyDiff, readOpsDiff, s.currentBdevIostat.TickRate)
		writeLatency := calculateLatencyInNs(writeLatencyDiff, writeOpsDiff, s.currentBdevIostat.TickRate)

		s.bdevMetricMap[bdev.Name] = &spdkrpc.Metrics{
			ReadIOPS:        uint64(float64(readOpsDiff) / elapsedSeconds),
			WriteIOPS:       uint64(float64(writeOpsDiff) / elapsedSeconds),
			ReadThroughput:  uint64(float64(bytesReadDiff) / elapsedSeconds),
			WriteThroughput: uint64(float64(bytesWrittenDiff) / elapsedSeconds),
			ReadLatency:     readLatency,
			WriteLatency:    writeLatency,
		}
	}
}

// Converts latency from ticks to nanoseconds
func calculateLatencyInNs(latencyTicks, opsDiff, tickRate uint64) uint64 {
	if opsDiff == 0 {
		return 0 // Prevent division by zero
	}
	return uint64((float64(latencyTicks) / float64(opsDiff)) * (1e9 / float64(tickRate)))
}

func (s *Server) MetricsGet(ctx context.Context, req *spdkrpc.MetricsRequest) (ret *spdkrpc.Metrics, err error) {
	s.RLock()
	defer s.RUnlock()
	m, ok := s.bdevMetricMap[req.Name]
	if !ok {
		return nil, grpcstatus.Errorf(grpccodes.NotFound, "cannot find engine metrics: %s", req.Name)
	}
	return m, nil
}

func setNvmeHotPlug(spdkClient *spdkclient.Client, enable bool) (success bool) {
	// default value: 100000 microseconds = 0.1 seconds
	_, hotPlugErr := spdkClient.BdevNvmeSetHotplug(enable, 100000)
	if hotPlugErr != nil {
		logrus.WithError(hotPlugErr).Warnf("Failed to set nvme hotplug to %v", enable)
		return false
	}
	return true
}

// engineFrontendByVolumeName returns the first engine frontend that matches
// the given volume name, or nil if none exists. Caller must hold s.RLock or
// s.Lock.
func (s *Server) engineFrontendByVolumeName(volumeName string) *EngineFrontend {
	for _, ef := range s.engineFrontendMap {
		if ef.VolumeName == volumeName {
			return ef
		}
	}
	return nil
}

func toEngineFrontendCreateGRPCError(err error, format string, args ...any) error {
	code := grpccodes.Internal

	// Check sentinel errors first — they are the most specific indicators
	// of what went wrong and should take priority over any embedded gRPC
	// status that might exist deeper in the error chain.
	switch {
	case errors.Is(err, ErrEngineFrontendCreateInvalidArgument):
		code = grpccodes.InvalidArgument
	case errors.Is(err, ErrEngineFrontendCreatePrecondition):
		code = grpccodes.FailedPrecondition
	case errors.Is(err, context.DeadlineExceeded):
		code = grpccodes.DeadlineExceeded
	case errors.Is(err, context.Canceled):
		code = grpccodes.Canceled
	default:
		// Fall back to any embedded gRPC status.
		if statusErr, ok := grpcstatus.FromError(errors.UnwrapAll(err)); ok {
			code = statusErr.Code()
		}
	}

	return grpcstatus.Error(code, errors.Wrapf(err, format, args...).Error())
}

func toEngineFrontendLifecycleGRPCError(err error, format string, args ...any) error {
	code := grpccodes.Internal

	switch {
	case errors.Is(err, ErrEngineFrontendLifecyclePrecondition), errors.Is(err, ErrSwitchOverTargetPrecondition):
		code = grpccodes.FailedPrecondition
	case errors.Is(err, ErrEngineFrontendLifecycleUnimplemented):
		code = grpccodes.Unimplemented
	case errors.Is(err, context.DeadlineExceeded):
		code = grpccodes.DeadlineExceeded
	case errors.Is(err, context.Canceled):
		code = grpccodes.Canceled
	default:
		if statusErr, ok := grpcstatus.FromError(errors.UnwrapAll(err)); ok {
			code = statusErr.Code()
		}
	}

	return grpcstatus.Error(code, errors.Wrapf(err, format, args...).Error())
}

func toExpansionGRPCError(err error, format string, args ...interface{}) error {
	code := grpccodes.Internal

	// Preserve downstream gRPC code when available.
	if statusErr, ok := grpcstatus.FromError(errors.UnwrapAll(err)); ok {
		code = statusErr.Code()
	} else {
		switch {
		case errors.Is(err, ErrExpansionInProgress), errors.Is(err, ErrRestoringInProgress):
			code = grpccodes.FailedPrecondition
		case errors.Is(err, ErrExpansionInvalidSize):
			code = grpccodes.InvalidArgument
		case errors.Is(err, context.DeadlineExceeded):
			code = grpccodes.DeadlineExceeded
		case errors.Is(err, context.Canceled):
			code = grpccodes.Canceled
		}
	}

	return grpcstatus.Error(code, errors.Wrapf(err, format, args...).Error())
}

// GetEngineStruct returns the internal Engine struct.
// This is for testing purposes only to allow access to internal fields and methods not exposed via RPC.
func (s *Server) GetEngineStruct(name string) *Engine {
	s.RLock()
	defer s.RUnlock()
	return s.engineMap[name]
}

// GetReplicaStruct returns the internal Replica struct.
// This is for testing purposes only to allow tests to inspect or manipulate internal replica state.
func (s *Server) GetReplicaStruct(name string) *Replica {
	s.RLock()
	defer s.RUnlock()
	return s.replicaMap[name]
}

// recoverEngineFrontends loads persisted engine frontend records from disk
// and attempts to recover them by detecting existing NVMe initiators on the host.
// This is called during server startup to restore state after instance-manager restart.
func (s *Server) recoverEngineFrontends() {
	if s.metadataDir == "" {
		return
	}

	records, err := loadEngineFrontendRecords(s.metadataDir)
	if err != nil {
		logrus.WithError(err).Error("Failed to load engine frontend records for recovery")
		return
	}

	if len(records) == 0 {
		return
	}

	logrus.Infof("Recovering %d engine frontend(s) from persisted records", len(records))

	s.Lock()
	spdkClient := s.spdkClient
	for _, record := range records {
		if _, exists := s.engineFrontendMap[record.Name]; exists {
			logrus.Infof("Engine frontend %s already exists in map, skipping recovery", record.Name)
			continue
		}

		ef := NewEngineFrontend(record.Name, record.EngineName, record.VolumeName,
			record.Frontend, record.SpecSize, 0, 0, s.updateChs[types.InstanceTypeEngineFrontend])
		ef.metadataDir = s.metadataDir
		if ef.NvmeTcpFrontend != nil {
			if record.TargetIP != "" {
				ef.NvmeTcpFrontend.TargetIP = record.TargetIP
				ef.EngineIP = record.TargetIP
			}
			if record.TargetPort != 0 {
				ef.NvmeTcpFrontend.TargetPort = record.TargetPort
			}
		}

		s.engineFrontendMap[record.Name] = ef

		logrus.Infof("Recovered engine frontend %s for volume %s from persisted record", record.Name, record.VolumeName)
	}
	s.Unlock()

	// Attempt to recover each frontend's initiator state from the host.
	// This is done outside the server lock to avoid holding it during potentially
	// slow NVMe device discovery operations.
	for _, record := range records {
		s.RLock()
		ef := s.engineFrontendMap[record.Name]
		s.RUnlock()

		if ef == nil {
			continue
		}

		if err := ef.RecoverFromHost(spdkClient); err != nil {
			if errors.Is(err, ErrRecoverDeviceNotFound) {
				logrus.Warnf("Removing engine frontend %s from map: device not found on host", record.Name)
			} else {
				logrus.WithError(err).Warnf("Removing engine frontend %s from map: recovery failed", record.Name)
			}

			// Properly shut down the frontend instance (close stopCh,
			// clean up any partially-recovered initiator, remove the
			// persisted record) before removing it from the map.
			// This follows the same pattern as the race-loser cleanup
			// in EngineFrontendCreate.
			if deleteErr := ef.Delete(spdkClient); deleteErr != nil {
				logrus.WithError(deleteErr).Warnf("Failed to clean up engine frontend %s during recovery removal", record.Name)
			}

			s.Lock()
			delete(s.engineFrontendMap, record.Name)
			s.Unlock()
		}
	}
}
</file>

<file path="pkg/spdk/snapshot_test.go">
package spdk

import (
	"fmt"
	"strings"

	"github.com/longhorn/longhorn-spdk-engine/pkg/client"
	"github.com/longhorn/longhorn-spdk-engine/pkg/types"

	. "gopkg.in/check.v1"
)

func (s *TestSuite) TestSnapshotOperationPreCheckCreateGeneratesName(c *C) {
	fmt.Println("Testing snapshotOperationPreCheckWithoutLock generates snapshot name for create operation")

	e := NewEngine("engine-a", "vol-a", types.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))

	snapshotName, err := e.snapshotOperationPreCheckWithoutLock(map[string]*client.SPDKClient{}, "", SnapshotOperationCreate)
	c.Assert(err, IsNil)
	c.Assert(snapshotName, Not(Equals), "")
	c.Assert(len(snapshotName), Equals, 8)
}

func (s *TestSuite) TestSnapshotOperationPreCheckDeleteEmptyName(c *C) {
	fmt.Println("Testing snapshotOperationPreCheckWithoutLock returns error for delete operation with empty snapshot name")

	e := NewEngine("engine-a", "vol-a", types.FrontendSPDKTCPBlockdev, 10, make(chan interface{}, 1))

	_, err := e.snapshotOperationPreCheckWithoutLock(map[string]*client.SPDKClient{}, "", SnapshotOperationDelete)
	c.Assert(err, NotNil)
	c.Assert(strings.Contains(err.Error(), "empty snapshot name"), Equals, true)
}

func (s *TestSuite) TestEngineFrontendSnapshotOperationCreateFailsWhenInitiatorNil(c *C) {
	fmt.Println("Testing engine frontend snapshotOperation returns error when initiator is nil for create operation")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", types.FrontendSPDKTCPBlockdev, 10, 0, 0, make(chan interface{}, 1))
	ef.State = types.InstanceStateRunning
	ef.Frontend = types.FrontendSPDKTCPBlockdev
	ef.Endpoint = "/dev/longhorn/test"
	ef.initiator = nil

	_, err := ef.snapshotOperation("snap-1", SnapshotOperationCreate, nil)
	c.Assert(err, NotNil)
	c.Assert(strings.Contains(err.Error(), "failed to suspend before the snapshot operation"), Equals, true)
	c.Assert(strings.Contains(err.Error(), "initiator is not initialized"), Equals, true)
}
</file>

<file path="pkg/spdk/switchover_test.go">
package spdk

import (
	"context"
	"fmt"
	"strings"
	"time"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/go-spdk-helper/pkg/initiator"
	"github.com/longhorn/types/pkg/generated/spdkrpc"

	helpertypes "github.com/longhorn/go-spdk-helper/pkg/types"

	lhtypes "github.com/longhorn/longhorn-spdk-engine/pkg/types"

	. "gopkg.in/check.v1"
)

func (s *TestSuite) TestEngineFrontendSwitchOverTargetNvmfSuccess(c *C) {
	fmt.Println("Testing EngineFrontend.SwitchOverTarget for SPDK TCP NVMe-oF frontend with successful switchover")

	updateCh := make(chan interface{}, 1)
	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPNvmf, 1024, 0, 0, updateCh)

	ef.State = lhtypes.InstanceStateRunning
	ef.NvmeTcpFrontend.Nqn = "nqn.2014-08.org.nvmexpress:uuid:test-a"
	ef.NvmeTcpFrontend.Nguid = "old-nguid"
	ef.EngineIP = "10.0.0.1"
	ef.NvmeTcpFrontend.TargetIP = "10.0.0.1"
	ef.NvmeTcpFrontend.TargetPort = 2000
	ef.Endpoint = GetNvmfEndpoint(ef.NvmeTcpFrontend.Nqn, ef.NvmeTcpFrontend.TargetIP, ef.NvmeTcpFrontend.TargetPort)

	err := ef.SwitchOverTarget(nil, "engine-b", "10.0.0.2:3000")
	c.Assert(err, IsNil)
	c.Assert(ef.EngineName, Equals, "engine-b")
	c.Assert(ef.EngineIP, Equals, "10.0.0.2")
	c.Assert(ef.NvmeTcpFrontend.TargetIP, Equals, "10.0.0.2")
	c.Assert(ef.NvmeTcpFrontend.TargetPort, Equals, int32(3000))

	expectedNQN := helpertypes.GetNQN("engine-b")
	c.Assert(ef.NvmeTcpFrontend.Nqn, Equals, expectedNQN)

	expectedEndpoint := GetNvmfEndpoint(expectedNQN, "10.0.0.2", 3000)
	c.Assert(ef.Endpoint, Equals, expectedEndpoint)

	select {
	case <-updateCh:
	default:
		c.Fatal("expected update notification after target switchover")
	}
}

func (s *TestSuite) TestEngineFrontendSwitchOverTargetBlockdevRequiresSuspended(c *C) {
	fmt.Println("Testing EngineFrontend.SwitchOverTarget for SPDK TCP Blockdev frontend requires suspended state")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1))
	ef.State = lhtypes.InstanceStateRunning

	err := ef.SwitchOverTarget(nil, "engine-b", "10.0.0.2:3000")
	c.Assert(err, NotNil)
	c.Assert(strings.Contains(err.Error(), "must be suspended"), Equals, true)
}

func (s *TestSuite) TestEngineFrontendSuspendIdempotent(c *C) {
	fmt.Println("Testing EngineFrontend.Suspend is idempotent when already suspended")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1))
	ef.State = lhtypes.InstanceStateSuspended

	err := ef.Suspend(nil)
	c.Assert(err, IsNil)
	c.Assert(string(ef.State), Equals, string(lhtypes.InstanceStateSuspended))
}

func (s *TestSuite) TestServerEngineFrontendSwitchOverLookupByEngineName(c *C) {
	fmt.Println("Testing Server.EngineFrontendSwitchOver lookup by engine name")

	updateCh := make(chan interface{}, 1)
	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPNvmf, 1024, 0, 0, updateCh)
	ef.State = lhtypes.InstanceStateRunning
	ef.NvmeTcpFrontend.Nqn = "nqn.2014-08.org.nvmexpress:uuid:test-a"

	srv := &Server{
		engineFrontendMap: map[string]*EngineFrontend{
			ef.Name: ef,
		},
	}

	_, err := srv.EngineFrontendSwitchOver(context.Background(), &spdkrpc.EngineFrontendSwitchOverRequest{
		Name:          ef.EngineName,
		EngineName:    "engine-b",
		TargetAddress: "10.0.0.2:3000",
	})
	c.Assert(err, IsNil)

	c.Assert(ef.NvmeTcpFrontend.TargetIP, Equals, "10.0.0.2")
	c.Assert(ef.NvmeTcpFrontend.TargetPort, Equals, int32(3000))
	c.Assert(ef.EngineName, Equals, "engine-b")
}

func (s *TestSuite) TestServerEngineFrontendSwitchOverAmbiguousEngineName(c *C) {
	fmt.Println("Testing Server.EngineFrontendSwitchOver with ambiguous engine name")

	srv := &Server{
		engineFrontendMap: map[string]*EngineFrontend{
			"ef-a": NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPNvmf, 1024, 0, 0, make(chan interface{}, 1)),
			"ef-b": NewEngineFrontend("ef-b", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPNvmf, 1024, 0, 0, make(chan interface{}, 1)),
		},
	}

	_, err := srv.EngineFrontendSwitchOver(context.Background(), &spdkrpc.EngineFrontendSwitchOverRequest{
		Name:          "engine-a",
		EngineName:    "engine-b",
		TargetAddress: "10.0.0.2:3000",
	})
	c.Assert(err, NotNil)

	st, ok := grpcstatus.FromError(err)
	c.Assert(ok, Equals, true)
	c.Assert(st.Code(), Equals, grpccodes.FailedPrecondition)
}

func (s *TestSuite) TestServerEngineFrontendSwitchOverInvalidAddress(c *C) {
	fmt.Println("Testing Server.EngineFrontendSwitchOver with invalid target address")

	srv := &Server{
		engineFrontendMap: map[string]*EngineFrontend{},
	}

	_, err := srv.EngineFrontendSwitchOver(context.Background(), &spdkrpc.EngineFrontendSwitchOverRequest{
		Name:          "ef-a",
		EngineName:    "engine-b",
		TargetAddress: "10.0.0.2",
	})
	c.Assert(err, NotNil)

	st, ok := grpcstatus.FromError(err)
	c.Assert(ok, Equals, true)
	c.Assert(st.Code(), Equals, grpccodes.InvalidArgument)
}

func (s *TestSuite) TestServerEngineFrontendSwitchOverBlockdevRequiresSuspended(c *C) {
	fmt.Println("Testing Server.EngineFrontendSwitchOver for blockdev frontend requires suspended state")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1))
	ef.State = lhtypes.InstanceStateRunning

	srv := &Server{
		engineFrontendMap: map[string]*EngineFrontend{
			ef.Name: ef,
		},
	}

	_, err := srv.EngineFrontendSwitchOver(context.Background(), &spdkrpc.EngineFrontendSwitchOverRequest{
		Name:          ef.Name,
		EngineName:    "engine-b",
		TargetAddress: "10.0.0.2:3000",
	})
	c.Assert(err, NotNil)

	st, ok := grpcstatus.FromError(err)
	c.Assert(ok, Equals, true)
	c.Assert(st.Code(), Equals, grpccodes.FailedPrecondition)
}

func (s *TestSuite) TestServerEngineFrontendSwitchOverRejectedDuringRestore(c *C) {
	fmt.Println("Testing Server.EngineFrontendSwitchOver is rejected during restore")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPNvmf, 1024, 0, 0, make(chan interface{}, 1))
	ef.State = lhtypes.InstanceStateRunning
	ef.IsRestoring = true

	srv := &Server{
		engineFrontendMap: map[string]*EngineFrontend{
			ef.Name: ef,
		},
	}

	_, err := srv.EngineFrontendSwitchOver(context.Background(), &spdkrpc.EngineFrontendSwitchOverRequest{
		Name:          ef.Name,
		EngineName:    "engine-b",
		TargetAddress: "10.0.0.2:3000",
	})
	c.Assert(err, NotNil)

	st, ok := grpcstatus.FromError(err)
	c.Assert(ok, Equals, true)
	c.Assert(st.Code(), Equals, grpccodes.FailedPrecondition)
}

func (s *TestSuite) TestServerEngineFrontendSwitchOverRejectedDuringExpand(c *C) {
	fmt.Println("Testing Server.EngineFrontendSwitchOver is rejected during expansion")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPNvmf, 1024, 0, 0, make(chan interface{}, 1))
	ef.State = lhtypes.InstanceStateRunning
	ef.isExpanding = true

	srv := &Server{
		engineFrontendMap: map[string]*EngineFrontend{
			ef.Name: ef,
		},
	}

	_, err := srv.EngineFrontendSwitchOver(context.Background(), &spdkrpc.EngineFrontendSwitchOverRequest{
		Name:          ef.Name,
		EngineName:    "engine-b",
		TargetAddress: "10.0.0.2:3000",
	})
	c.Assert(err, NotNil)

	st, ok := grpcstatus.FromError(err)
	c.Assert(ok, Equals, true)
	c.Assert(st.Code(), Equals, grpccodes.FailedPrecondition)
}

func (s *TestSuite) TestEngineFrontendSwitchOverTargetResolveEngineNameFallback(c *C) {
	fmt.Println("Testing EngineFrontend.SwitchOverTarget for SPDK TCP NVMe-oF frontend with engine name fallback")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPNvmf, 1024, 0, 0, make(chan interface{}, 1))
	ef.State = lhtypes.InstanceStateRunning
	ef.resolveEngineNameByTargetAddressFn = func(targetAddress string) (string, error) {
		return "engine-c", nil
	}

	err := ef.SwitchOverTarget(nil, "", "10.0.0.2:3000")
	c.Assert(err, IsNil)
	c.Assert(ef.EngineName, Equals, "engine-c")
}

func (s *TestSuite) TestEngineFrontendSwitchOverTargetBlockdevNoOpWithoutSuspend(c *C) {
	fmt.Println("Testing EngineFrontend.SwitchOverTarget for SPDK TCP Blockdev frontend with no-op switchover without suspend")

	updateCh := make(chan interface{}, 1)
	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, updateCh)
	ef.State = lhtypes.InstanceStateRunning
	ef.EngineIP = "10.0.0.1"
	ef.NvmeTcpFrontend.TargetIP = "10.0.0.1"
	ef.NvmeTcpFrontend.TargetPort = 2000
	ef.NvmeTcpFrontend.Nqn = helpertypes.GetNQN("engine-a")
	ef.NvmeTcpFrontend.Nguid = generateNGUID("engine-a")
	ef.Endpoint = "/dev/longhorn/vol-a"

	resolveCalled := false
	ef.resolveEngineNameByTargetAddressFn = func(targetAddress string) (string, error) {
		resolveCalled = true
		return "", fmt.Errorf("should not resolve engine name for no-op switchover")
	}

	err := ef.SwitchOverTarget(nil, "", "10.0.0.1:2000")
	c.Assert(err, IsNil)
	c.Assert(resolveCalled, Equals, false)

	select {
	case <-updateCh:
		c.Fatal("did not expect update notification for no-op switchover")
	default:
	}
}

func (s *TestSuite) TestEngineFrontendSwitchOverTargetBlockdevRollbackSuccess(c *C) {
	fmt.Println("Testing EngineFrontend.SwitchOverTarget for SPDK TCP Blockdev frontend with successful rollback")

	updateCh := make(chan interface{}, 1)
	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, updateCh)
	ef.State = lhtypes.InstanceStateSuspended

	oldEngineName := "engine-a"
	oldEngineIP := "10.0.0.1"
	oldTargetIP := "10.0.0.1"
	oldTargetPort := int32(2000)
	oldNQN := helpertypes.GetNQN(oldEngineName)
	oldNGUID := generateNGUID(oldEngineName)
	oldEndpoint := "/dev/longhorn/vol-a"

	ef.EngineName = oldEngineName
	ef.EngineIP = oldEngineIP
	ef.NvmeTcpFrontend.TargetIP = oldTargetIP
	ef.NvmeTcpFrontend.TargetPort = oldTargetPort
	ef.NvmeTcpFrontend.Nqn = oldNQN
	ef.NvmeTcpFrontend.Nguid = oldNGUID
	ef.Endpoint = oldEndpoint
	ef.dmDeviceIsBusy = true
	ef.initiator = &initiator.Initiator{
		Endpoint:    oldEndpoint,
		NVMeTCPInfo: &initiator.NVMeTCPInfo{SubsystemNQN: oldNQN},
	}

	var callTargets []string
	ef.startNvmeTCPInitiatorFn = func(transportAddress, transportServiceID string, dmDeviceAndEndpointCleanupRequired bool, stop bool) (bool, error) {
		callTargets = append(callTargets, transportAddress+":"+transportServiceID)
		if len(callTargets) == 1 {
			return false, fmt.Errorf("switch failed")
		}
		return true, nil
	}

	err := ef.SwitchOverTarget(nil, "engine-b", "10.0.0.2:3000")
	c.Assert(err, NotNil)
	c.Assert(strings.Contains(err.Error(), "switch failed"), Equals, true)

	c.Assert(len(callTargets), Equals, 2)
	c.Assert(callTargets[0], Equals, "10.0.0.2:3000")
	c.Assert(callTargets[1], Equals, "10.0.0.1:2000")

	c.Assert(ef.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateSuspended))
	c.Assert(ef.EngineName, Equals, oldEngineName)
	c.Assert(ef.EngineIP, Equals, oldEngineIP)
	c.Assert(ef.NvmeTcpFrontend.TargetIP, Equals, oldTargetIP)
	c.Assert(ef.NvmeTcpFrontend.TargetPort, Equals, oldTargetPort)
	c.Assert(ef.NvmeTcpFrontend.Nqn, Equals, oldNQN)
	c.Assert(ef.NvmeTcpFrontend.Nguid, Equals, oldNGUID)
	c.Assert(ef.Endpoint, Equals, oldEndpoint)
	c.Assert(ef.initiator.NVMeTCPInfo, NotNil)
	c.Assert(ef.initiator.NVMeTCPInfo.SubsystemNQN, Equals, oldNQN)

	select {
	case <-updateCh:
	default:
		c.Fatal("expected update notification after rollback")
	}
}

func (s *TestSuite) TestEngineFrontendSwitchOverTargetBlockdevRollbackFailure(c *C) {
	fmt.Println("Testing EngineFrontend.SwitchOverTarget for SPDK TCP Blockdev frontend with failed rollback")

	updateCh := make(chan interface{}, 1)
	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, updateCh)
	ef.State = lhtypes.InstanceStateSuspended
	ef.EngineIP = "10.0.0.1"
	ef.NvmeTcpFrontend.TargetIP = "10.0.0.1"
	ef.NvmeTcpFrontend.TargetPort = 2000
	ef.NvmeTcpFrontend.Nqn = helpertypes.GetNQN("engine-a")
	ef.NvmeTcpFrontend.Nguid = generateNGUID("engine-a")
	ef.initiator = &initiator.Initiator{
		NVMeTCPInfo: &initiator.NVMeTCPInfo{SubsystemNQN: ef.NvmeTcpFrontend.Nqn},
	}

	callCount := 0
	ef.startNvmeTCPInitiatorFn = func(transportAddress, transportServiceID string, dmDeviceAndEndpointCleanupRequired bool, stop bool) (bool, error) {
		callCount++
		if callCount == 1 {
			return false, fmt.Errorf("switch failed")
		}
		return false, fmt.Errorf("rollback failed")
	}

	err := ef.SwitchOverTarget(nil, "engine-b", "10.0.0.2:3000")
	c.Assert(err, NotNil)
	c.Assert(strings.Contains(err.Error(), "switch failed"), Equals, true)
	c.Assert(strings.Contains(err.Error(), "rollback failed"), Equals, true)
	c.Assert(ef.State, Equals, lhtypes.InstanceState(lhtypes.InstanceStateError))

	select {
	case <-updateCh:
	default:
		c.Fatal("expected update notification after rollback failure")
	}
}

func (s *TestSuite) TestEngineFrontendSwitchOverTargetBlockdevInProgressGuard(c *C) {
	fmt.Println("Testing EngineFrontend.SwitchOverTarget in-progress guard for SPDK TCP Blockdev frontend")

	updateCh := make(chan interface{}, 2)
	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, updateCh)
	ef.State = lhtypes.InstanceStateSuspended
	ef.EngineIP = "10.0.0.1"
	ef.NvmeTcpFrontend.TargetIP = "10.0.0.1"
	ef.NvmeTcpFrontend.TargetPort = 2000
	ef.NvmeTcpFrontend.Nqn = helpertypes.GetNQN("engine-a")
	ef.NvmeTcpFrontend.Nguid = generateNGUID("engine-a")
	ef.initiator = &initiator.Initiator{NVMeTCPInfo: &initiator.NVMeTCPInfo{SubsystemNQN: ef.NvmeTcpFrontend.Nqn}}
	ef.getInitiatorEndpointFn = func() string { return "/dev/longhorn/vol-a" }

	enteredCh := make(chan struct{}, 1)
	releaseCh := make(chan struct{})
	ef.startNvmeTCPInitiatorFn = func(transportAddress, transportServiceID string, dmDeviceAndEndpointCleanupRequired bool, stop bool) (bool, error) {
		enteredCh <- struct{}{}
		<-releaseCh
		return false, nil
	}

	firstErrCh := make(chan error, 1)
	go func() {
		firstErrCh <- ef.SwitchOverTarget(nil, "engine-b", "10.0.0.2:3000")
	}()

	select {
	case <-enteredCh:
	case <-time.After(2 * time.Second):
		c.Fatal("timeout waiting for first switchover to enter phase-2")
	}

	// While first switchover is in phase-2, read operations should remain responsive.
	getDone := make(chan struct{}, 1)
	go func() {
		_ = ef.Get()
		getDone <- struct{}{}
	}()
	select {
	case <-getDone:
	case <-time.After(1 * time.Second):
		c.Fatal("Get() blocked while switchover is in progress")
	}

	// Concurrent switchover should be rejected by the in-progress guard.
	err := ef.SwitchOverTarget(nil, "engine-c", "10.0.0.3:3000")
	c.Assert(err, NotNil)
	c.Assert(strings.Contains(err.Error(), "already in progress"), Equals, true)

	close(releaseCh)
	select {
	case err := <-firstErrCh:
		c.Assert(err, IsNil)
	case <-time.After(2 * time.Second):
		c.Fatal("timeout waiting for first switchover to complete")
	}
}

func (s *TestSuite) TestEngineFrontendDeleteRejectedDuringSwitchOver(c *C) {
	fmt.Println("Testing EngineFrontend.Delete is rejected while switch over is in progress")

	updateCh := make(chan interface{}, 2)
	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, updateCh)
	ef.State = lhtypes.InstanceStateSuspended
	ef.EngineIP = "10.0.0.1"
	ef.NvmeTcpFrontend.TargetIP = "10.0.0.1"
	ef.NvmeTcpFrontend.TargetPort = 2000
	ef.NvmeTcpFrontend.Nqn = helpertypes.GetNQN("engine-a")
	ef.NvmeTcpFrontend.Nguid = generateNGUID("engine-a")
	ef.initiator = &initiator.Initiator{NVMeTCPInfo: &initiator.NVMeTCPInfo{SubsystemNQN: ef.NvmeTcpFrontend.Nqn}}
	ef.getInitiatorEndpointFn = func() string { return "/dev/longhorn/vol-a" }

	enteredCh := make(chan struct{}, 1)
	releaseCh := make(chan struct{})
	ef.startNvmeTCPInitiatorFn = func(transportAddress, transportServiceID string, dmDeviceAndEndpointCleanupRequired bool, stop bool) (bool, error) {
		enteredCh <- struct{}{}
		<-releaseCh
		return false, nil
	}

	switchErrCh := make(chan error, 1)
	go func() {
		switchErrCh <- ef.SwitchOverTarget(nil, "engine-b", "10.0.0.2:3000")
	}()

	select {
	case <-enteredCh:
	case <-time.After(2 * time.Second):
		c.Fatal("timeout waiting for switchover to enter phase-2")
	}

	deleteErrCh := make(chan error, 1)
	go func() {
		deleteErrCh <- ef.Delete(nil)
	}()

	select {
	case err := <-deleteErrCh:
		c.Assert(err, NotNil)
		c.Assert(strings.Contains(err.Error(), "switching over target"), Equals, true)
	case <-time.After(1 * time.Second):
		c.Fatal("Delete() blocked while switchover is in progress")
	}

	close(releaseCh)
	select {
	case err := <-switchErrCh:
		c.Assert(err, IsNil)
	case <-time.After(2 * time.Second):
		c.Fatal("timeout waiting for switchover to complete")
	}
}

func (s *TestSuite) TestEngineFrontendSwitchOverTargetRejectedDuringExpand(c *C) {
	fmt.Println("Testing EngineFrontend.SwitchOverTarget is rejected while expansion is in progress")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPNvmf, 1024, 0, 0, make(chan interface{}, 1))
	ef.State = lhtypes.InstanceStateRunning
	ef.isExpanding = true

	err := ef.SwitchOverTarget(nil, "engine-b", "10.0.0.2:3000")
	c.Assert(err, NotNil)
	c.Assert(strings.Contains(err.Error(), "expansion is in progress"), Equals, true)
}

func (s *TestSuite) TestEngineFrontendSwitchOverTargetRejectedDuringRestore(c *C) {
	fmt.Println("Testing EngineFrontend.SwitchOverTarget is rejected while restore is in progress")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPNvmf, 1024, 0, 0, make(chan interface{}, 1))
	ef.State = lhtypes.InstanceStateRunning
	ef.IsRestoring = true

	err := ef.SwitchOverTarget(nil, "engine-b", "10.0.0.2:3000")
	c.Assert(err, NotNil)
	c.Assert(strings.Contains(err.Error(), "restore is in progress"), Equals, true)
}

func (s *TestSuite) TestServerEngineFrontendSwitchOverEngineNotFound(c *C) {
	fmt.Println("Testing Server.EngineFrontendSwitchOver with target engine not found")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPNvmf, 1024, 0, 0, make(chan interface{}, 1))
	ef.State = lhtypes.InstanceStateRunning
	ef.resolveEngineNameByTargetAddressFn = func(targetAddress string) (string, error) {
		return "", ErrSwitchOverTargetEngineNotFound
	}

	srv := &Server{
		engineFrontendMap: map[string]*EngineFrontend{
			ef.Name: ef,
		},
	}

	_, err := srv.EngineFrontendSwitchOver(context.Background(), &spdkrpc.EngineFrontendSwitchOverRequest{
		Name:          ef.Name,
		TargetAddress: "10.0.0.2:3000",
	})
	c.Assert(err, NotNil)

	st, ok := grpcstatus.FromError(err)
	c.Assert(ok, Equals, true)
	c.Assert(st.Code(), Equals, grpccodes.NotFound)
}

func (s *TestSuite) TestCreateUblkFrontendNilReturnsCorrectErrorField(c *C) {
	fmt.Println("Testing createUblkFrontend with nil UblkFrontend returns error referencing UblkFrontend")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendUBLK, 1024, 0, 0, make(chan interface{}, 1))
	ef.UblkFrontend = nil // force nil

	err := ef.createUblkFrontend(nil)
	c.Assert(err, NotNil)
	c.Assert(strings.Contains(err.Error(), "UblkFrontend"), Equals, true)
	// Ensure it does NOT reference the wrong field
	c.Assert(strings.Contains(err.Error(), "NvmeTcpFrontend"), Equals, false)
}

func (s *TestSuite) TestIsInitiatorCreationRequiredUblkReturnsTrue(c *C) {
	fmt.Println("Testing isInitiatorCreationRequired returns true for UBLK frontend")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendUBLK, 1024, 0, 0, make(chan interface{}, 1))

	required, err := ef.isInitiatorCreationRequired("10.0.0.1")
	c.Assert(err, IsNil)
	c.Assert(required, Equals, true)
}

func (s *TestSuite) TestIsInitiatorCreationRequiredNvmeTcpBlockdevNewEngine(c *C) {
	fmt.Println("Testing isInitiatorCreationRequired returns true for new NVMe/TCP blockdev engine (port=0)")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1))

	required, err := ef.isInitiatorCreationRequired("10.0.0.1")
	c.Assert(err, IsNil)
	c.Assert(required, Equals, true)
}

func (s *TestSuite) TestIsInitiatorCreationRequiredNvmeTcpBlockdevExistingEngine(c *C) {
	fmt.Println("Testing isInitiatorCreationRequired returns false for existing NVMe/TCP blockdev engine (port!=0)")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1))
	ef.NvmeTcpFrontend.TargetPort = 3000

	required, err := ef.isInitiatorCreationRequired("10.0.0.1")
	c.Assert(err, IsNil)
	c.Assert(required, Equals, false)
}

func (s *TestSuite) TestIsInitiatorCreationRequiredNilNvmeTcpFrontendReturnsError(c *C) {
	fmt.Println("Testing isInitiatorCreationRequired returns error when NvmeTcpFrontend is nil for non-UBLK frontend")

	ef := NewEngineFrontend("ef-a", "engine-a", "vol-a", lhtypes.FrontendSPDKTCPBlockdev, 1024, 0, 0, make(chan interface{}, 1))
	ef.NvmeTcpFrontend = nil

	_, err := ef.isInitiatorCreationRequired("10.0.0.1")
	c.Assert(err, NotNil)
	c.Assert(strings.Contains(err.Error(), "invalid NvmeTcpFrontend"), Equals, true)
}
</file>

<file path="pkg/spdk/types.go">
package spdk

import (
	"encoding/hex"
	"fmt"
	"net"
	"regexp"
	"strconv"
	"strings"
	"sync"
	"time"

	"github.com/cockroachdb/errors"
	"github.com/google/uuid"

	"github.com/longhorn/types/pkg/generated/spdkrpc"

	spdkclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
	spdktypes "github.com/longhorn/go-spdk-helper/pkg/spdk/types"

	"github.com/longhorn/longhorn-spdk-engine/pkg/client"
	"github.com/longhorn/longhorn-spdk-engine/pkg/types"
	"github.com/longhorn/longhorn-spdk-engine/pkg/util"
)

const (
	DiskTypeFilesystem = "filesystem"
	DiskTypeBlock      = "block"

	ReplicaRebuildingLvolSuffix  = "rebuilding"
	ReplicaExpiredLvolSuffix     = "expired"
	ReplicaCloningLvolSuffix     = "cloning"
	RebuildingSnapshotNamePrefix = "rebuild"

	SyncTimeout = 60 * time.Minute

	maxRetries    = 30
	retryInterval = 1 * time.Second

	disconnectMaxRetries    = 5
	disconnectRetryInterval = 1 * time.Second

	MaxShallowCopyWaitTime   = 72 * time.Hour
	ShallowCopyCheckInterval = 3 * time.Second

	MaxSnapshotCloneWaitTime         = 72 * time.Hour
	SnapshotCloneStatusCheckInterval = 3 * time.Second
)

const (
	// Timeouts for RAID base bdev (replica)
	// The ctrlr_loss_timeout_sec setting applies to the base bdev's NVMe controller
	// and defines the timeout duration (30 seconds) for SPDK to attempt reconnecting to the controller
	// after losing connection.
	//
	// When an instance manager containing a replica is deleted, SPDK starts to reconnect to the base bdev's controller.
	// If the connection cannot be reestablished within the ctrlr_loss_timeout_sec period, the base bdev is removed from the RAID bdev.
	//
	// Because the ctrl-loss-tmo for the NVMe/TCP initiator connecting to the RAID target is also set to 30 seconds,
	// replicaCtrlrLossTimeoutSec and replicaFastIOFailTimeoutSec are set to 15 seconds and 10 seconds, respectively.
	//
	// If an I/O operation to a replica (base bdev) is unresponsive within 10 seconds, an I/O error is returned,
	// and the base bdev is deleted after 5 seconds.
	replicaCtrlrLossTimeoutSec  = 15
	replicaReconnectDelaySec    = 2
	replicaFastIOFailTimeoutSec = 10
	replicaTransportAckTimeout  = 10
	replicaKeepAliveTimeoutMs   = 10000
	replicaMultipath            = "disable"
)

var (
	// ErrEngineFrontendCreateInvalidArgument indicates the create request carries
	// invalid input, such as an unparsable target address.
	ErrEngineFrontendCreateInvalidArgument = errors.New("engine frontend create invalid argument")
	// ErrEngineFrontendCreatePrecondition indicates the frontend is not in a
	// state that can satisfy create preconditions.
	ErrEngineFrontendCreatePrecondition = errors.New("engine frontend create precondition failed")
	// ErrEngineFrontendLifecyclePrecondition indicates suspend/resume/delete
	// cannot proceed because the frontend is in an incompatible state.
	ErrEngineFrontendLifecyclePrecondition = errors.New("engine frontend lifecycle precondition failed")
	// ErrEngineFrontendLifecycleUnimplemented indicates the requested lifecycle
	// operation is not implemented for the current frontend type.
	ErrEngineFrontendLifecycleUnimplemented = errors.New("engine frontend lifecycle unimplemented")
)

var (
	// ErrRecoverDeviceNotFound indicates the NVMe device was not found on the
	// host during recovery. The persisted record should be removed.
	ErrRecoverDeviceNotFound = errors.New("device not found on host during recovery")
)

var (
	// ErrSwitchOverTargetInvalidInput indicates invalid user input for a target switchover request.
	ErrSwitchOverTargetInvalidInput = errors.New("invalid switchover target request")
	// ErrSwitchOverTargetPrecondition indicates the current frontend state cannot satisfy switchover preconditions.
	ErrSwitchOverTargetPrecondition = errors.New("switchover target precondition failed")
	// ErrSwitchOverTargetEngineNotFound indicates no engine can be resolved from the target side.
	ErrSwitchOverTargetEngineNotFound = errors.New("cannot find target engine for switchover")
	// ErrSwitchOverTargetInternal indicates switchover execution failed due to runtime/internal reasons.
	ErrSwitchOverTargetInternal = errors.New("failed to switch over target")
)

var (
	// ErrExpansionInProgress indicates expansion cannot proceed because another
	// expansion operation is already running.
	ErrExpansionInProgress = errors.New("expansion is in progress")
	// ErrRestoringInProgress indicates expansion cannot proceed while restoring.
	ErrRestoringInProgress = errors.New("restoring is in progress")
	// ErrExpansionInvalidSize indicates an invalid target size for expansion.
	ErrExpansionInvalidSize = errors.New("invalid expansion size")
)

type Lvol struct {
	sync.RWMutex

	Name       string
	UUID       string
	Alias      string
	SpecSize   uint64
	ActualSize uint64
	// Parent is the snapshot lvol name. <snapshot lvol name> consists of `<replica name>-snap-<snapshot name>`
	Parent string
	// Children is map[<snapshot lvol name>] rather than map[<snapshot name>]. <snapshot lvol name> consists of `<replica name>-snap-<snapshot name>`
	Children          map[string]*Lvol
	CreationTime      string
	UserCreated       bool
	SnapshotTimestamp string
	SnapshotChecksum  string
}

func ServiceBackingImageLvolToProtoBackingImageLvol(lvol *Lvol) *spdkrpc.Lvol {
	lvol.RLock()
	defer lvol.RUnlock()

	res := &spdkrpc.Lvol{
		Uuid:       lvol.UUID,
		Name:       lvol.Name,
		SpecSize:   lvol.SpecSize,
		ActualSize: lvol.ActualSize,
		// BackingImage has no parent
		Parent:       "",
		Children:     map[string]bool{},
		CreationTime: lvol.CreationTime,
		UserCreated:  false,
		// Use creation time instead
		SnapshotTimestamp: "",
	}

	for childLvolName := range lvol.Children {
		// For backing image, the children is map[<snapshot lvol name>]
		res.Children[childLvolName] = true
	}

	return res
}

func ServiceLvolToProtoLvol(replicaName string, lvol *Lvol) *spdkrpc.Lvol {
	if lvol == nil {
		return nil
	}
	res := &spdkrpc.Lvol{
		Uuid:              lvol.UUID,
		SpecSize:          lvol.SpecSize,
		ActualSize:        lvol.ActualSize,
		Parent:            GetSnapshotNameFromReplicaSnapshotLvolName(replicaName, lvol.Parent),
		Children:          map[string]bool{},
		CreationTime:      lvol.CreationTime,
		UserCreated:       lvol.UserCreated,
		SnapshotTimestamp: lvol.SnapshotTimestamp,
		SnapshotChecksum:  lvol.SnapshotChecksum,
	}

	if lvol.Name == replicaName {
		res.Name = types.VolumeHead
	} else {
		res.Name = GetSnapshotNameFromReplicaSnapshotLvolName(replicaName, lvol.Name)
	}

	for childLvolName := range lvol.Children {
		// spdkrpc.Lvol.Children is map[<snapshot name>] rather than map[<snapshot lvol name>]
		if childLvolName == replicaName {
			res.Children[types.VolumeHead] = true
		} else {
			res.Children[GetSnapshotNameFromReplicaSnapshotLvolName(replicaName, childLvolName)] = true
		}
	}

	return res
}

func BdevLvolInfoToServiceLvol(bdev *spdktypes.BdevInfo) *Lvol {
	svcLvol := &Lvol{
		Name:              spdktypes.GetLvolNameFromAlias(bdev.Aliases[0]),
		Alias:             bdev.Aliases[0],
		UUID:              bdev.UUID,
		SpecSize:          bdev.NumBlocks * uint64(bdev.BlockSize),
		ActualSize:        bdev.DriverSpecific.Lvol.NumAllocatedClusters * defaultClusterSize,
		Parent:            bdev.DriverSpecific.Lvol.BaseSnapshot,
		Children:          map[string]*Lvol{},
		CreationTime:      bdev.CreationTime,
		UserCreated:       bdev.DriverSpecific.Lvol.Xattrs[spdkclient.UserCreated] == strconv.FormatBool(true),
		SnapshotTimestamp: bdev.DriverSpecific.Lvol.Xattrs[spdkclient.SnapshotTimestamp],
		SnapshotChecksum:  bdev.DriverSpecific.Lvol.Xattrs[spdkclient.SnapshotChecksum],
	}

	// Need to further update this separately
	for _, childLvolName := range bdev.DriverSpecific.Lvol.Clones {
		svcLvol.Children[childLvolName] = nil
	}

	return svcLvol
}

func IsProbablyReplicaName(name string) bool {
	matched, _ := regexp.MatchString("^.+-r-[a-zA-Z0-9]{8}$", name)
	return matched
}

func GetBackingImageSnapLvolName(backingImageName string, lvsUUID string) string {
	return fmt.Sprintf("bi-%s-disk-%s", backingImageName, lvsUUID)
}

func GetBackingImageTempHeadLvolName(backingImageName string, lvsUUID string) string {
	return fmt.Sprintf("bi-%s-disk-%s-temp-head", backingImageName, lvsUUID)
}

func GetReplicaSnapshotLvolNamePrefix(replicaName string) string {
	return fmt.Sprintf("%s-snap-", replicaName)
}

func GetReplicaSnapshotLvolName(replicaName, snapshotName string) string {
	return fmt.Sprintf("%s%s", GetReplicaSnapshotLvolNamePrefix(replicaName), snapshotName)
}

func GetSnapshotNameFromReplicaSnapshotLvolName(replicaName, snapLvolName string) string {
	return strings.TrimPrefix(snapLvolName, GetReplicaSnapshotLvolNamePrefix(replicaName))
}

func IsReplicaLvol(replicaName, lvolName string) bool {
	return strings.HasPrefix(lvolName, fmt.Sprintf("%s-", replicaName)) || lvolName == replicaName
}

func IsReplicaSnapshotLvol(replicaName, lvolName string) bool {
	return strings.HasPrefix(lvolName, GetReplicaSnapshotLvolNamePrefix(replicaName))
}

func GenerateRebuildingSnapshotName() string {
	return fmt.Sprintf("%s-%s", RebuildingSnapshotNamePrefix, util.UUID()[:8])
}

func GenerateReplicaExpiredLvolName(replicaName string) string {
	return fmt.Sprintf("%s-%s-%s", replicaName, ReplicaExpiredLvolSuffix, util.UUID()[:8])
}

func GetReplicaRebuildingLvolName(replicaName string) string {
	return fmt.Sprintf("%s-%s", replicaName, ReplicaRebuildingLvolSuffix)
}

func IsRebuildingLvol(lvolName string) bool {
	return strings.HasSuffix(lvolName, ReplicaRebuildingLvolSuffix)
}

func IsReplicaExpiredLvol(replicaName, lvolName string) bool {
	return strings.HasPrefix(lvolName, fmt.Sprintf("%s-%s", replicaName, ReplicaExpiredLvolSuffix))
}

func GetReplicaNameFromRebuildingLvolName(lvolName string) string {
	return strings.TrimSuffix(lvolName, fmt.Sprintf("-%s", ReplicaRebuildingLvolSuffix))
}

func GetReplicaCloningLvolName(replicaName string) string {
	return fmt.Sprintf("%s-%s", replicaName, ReplicaCloningLvolSuffix)
}

func IsCloningLvol(lvolName string) bool {
	return strings.HasSuffix(lvolName, ReplicaCloningLvolSuffix)
}

func GetReplicaNameFromCloningLvolName(lvolName string) string {
	return strings.TrimSuffix(lvolName, fmt.Sprintf("-%s", ReplicaCloningLvolSuffix))
}

func GetTmpSnapNameForCloningLvol(replicaName string) string {
	return fmt.Sprintf("%s-%s-tmp", replicaName, ReplicaCloningLvolSuffix)
}

func GetNvmfEndpoint(nqn, ip string, port int32) string {
	return fmt.Sprintf("nvmf://%s:%d/%s", ip, port, nqn)
}

func GetServiceClient(address string) (*client.SPDKClient, error) {
	ip, _, err := net.SplitHostPort(address)
	if err != nil {
		return nil, err
	}
	// TODO: Can we use the fixed port
	addr := net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort))

	// TODO: Can we share the clients in the whole server?
	return client.NewSPDKClient(addr)
}

func GetBdevMap(cli *spdkclient.Client) (map[string]*spdktypes.BdevInfo, error) {
	bdevList, err := cli.BdevGetBdevs("", 0)
	if err != nil {
		return nil, err
	}

	bdevMap := map[string]*spdktypes.BdevInfo{}
	for idx := range bdevList {
		bdev := &bdevList[idx]
		bdevType := spdktypes.GetBdevType(bdev)

		switch bdevType {
		case spdktypes.BdevTypeLvol:
			if len(bdev.Aliases) != 1 {
				continue
			}
			bdevMap[bdev.Aliases[0]] = bdev
		case spdktypes.BdevTypeRaid:
			fallthrough
		default:
			bdevMap[bdev.Name] = bdev
		}
	}

	return bdevMap, nil
}

func GetBdevLvolMap(cli *spdkclient.Client) (map[string]*spdktypes.BdevInfo, error) {
	return GetBdevLvolMapWithFilter(cli, func(*spdktypes.BdevInfo) bool { return true })
}

func GetBdevLvolMapWithFilter(cli *spdkclient.Client, filter func(*spdktypes.BdevInfo) bool) (map[string]*spdktypes.BdevInfo, error) {
	bdevList, err := cli.BdevLvolGetWithFilter("", 0, filter)
	if err != nil {
		return nil, err
	}

	bdevLvolMap := map[string]*spdktypes.BdevInfo{}
	for idx := range bdevList {
		bdev := &bdevList[idx]
		bdevType := spdktypes.GetBdevType(bdev)
		if bdevType != spdktypes.BdevTypeLvol {
			continue
		}
		if len(bdev.Aliases) != 1 {
			continue
		}
		lvolName := spdktypes.GetLvolNameFromAlias(bdev.Aliases[0])
		bdevLvolMap[lvolName] = bdev
	}

	return bdevLvolMap, nil
}

func GetNvmfSubsystemMap(cli *spdkclient.Client) (map[string]*spdktypes.NvmfSubsystem, error) {
	subsystemList, err := cli.NvmfGetSubsystems("", "")
	if err != nil {
		return nil, err
	}

	subsystemMap := map[string]*spdktypes.NvmfSubsystem{}
	for idx := range subsystemList {
		subsystem := &subsystemList[idx]
		subsystemMap[subsystem.Nqn] = subsystem
	}

	return subsystemMap, nil
}

type BackupCreateInfo struct {
	BackupName     string
	IsIncremental  bool
	ReplicaAddress string
}

func generateNGUID(name string) string {
	nguid := uuid.NewSHA1(uuid.NameSpaceOID, []byte(name))
	return hex.EncodeToString(nguid[:]) // 32-char hex

}
</file>

<file path="pkg/spdk/util_test.go">
package spdk

import (
	"fmt"
	"testing"

	. "gopkg.in/check.v1"

	"github.com/longhorn/longhorn-spdk-engine/pkg/types"
)

func Test(t *testing.T) { TestingT(t) }

type TestSuite struct{}

var _ = Suite(&TestSuite{})

func (s *TestSuite) TestSplitHostPort(c *C) {
	fmt.Println("Testing splitHostPort with various address formats")

	type testCase struct {
		address      string
		expectedHost string
		expectedPort int32
		expectedErr  error
	}
	testCases := map[string]testCase{
		"splitHostPort(...): normal address": {
			address:      "127.0.0.1:8080",
			expectedHost: "127.0.0.1",
			expectedPort: 8080,
			expectedErr:  nil,
		},
		"splitHostPort(...): only host": {
			address:      "127.0.0.1",
			expectedHost: "127.0.0.1",
			expectedPort: 0,
			expectedErr:  nil,
		},
		"splitHostPort(...): only port": {
			address:      ":8080",
			expectedHost: "",
			expectedPort: 8080,
			expectedErr:  nil,
		},
		"splitHostPort(...): empty address": {
			address:      "",
			expectedHost: "",
			expectedPort: 0,
			expectedErr:  nil,
		},
	}
	for testName, testCase := range testCases {
		c.Logf("testing splitHostPort.%v", testName)

		hsot, port, err := splitHostPort(testCase.address)
		c.Assert(err, IsNil)
		c.Assert(hsot, Equals, testCase.expectedHost)
		c.Assert(port, Equals, testCase.expectedPort)
	}
}

func (s *TestSuite) TestExtractBackingImageAndDiskUUID(c *C) {
	fmt.Println("Testing ExtractBackingImageAndDiskUUID with various lvol name formats")

	type testCase struct {
		lvolName         string
		expectedBIName   string
		expectedDiskUUID string
		expectError      bool
	}
	testCases := map[string]testCase{
		"ExtractBackingImageAndDiskUUID(...): only disk with hyphens": {
			lvolName:         "bi-MyBackingImage-disk-12345-abcde",
			expectedBIName:   "MyBackingImage",
			expectedDiskUUID: "12345-abcde",
			expectError:      false,
		},
		"ExtractBackingImageAndDiskUUID(...): backing image name and disk with hyphens": {
			lvolName:         "bi-My-Backing-Image-disk-12345-abcde-xyz",
			expectedBIName:   "My-Backing-Image",
			expectedDiskUUID: "12345-abcde-xyz",
			expectError:      false,
		},
		"ExtractBackingImageAndDiskUUID(...): backing image name and disk without hyphens": {
			lvolName:         "bi-MyBackingImage-disk-12345",
			expectedBIName:   "MyBackingImage",
			expectedDiskUUID: "12345",
			expectError:      false,
		},
		"ExtractBackingImageAndDiskUUID(...):  doesn't start with bi- and doesn't contain -disk-": {
			lvolName:         "myWrongPattern",
			expectedBIName:   "",
			expectedDiskUUID: "",
			expectError:      true,
		},
		"ExtractBackingImageAndDiskUUID(...): doesn't have disk in the lvol name": {
			lvolName:         "bi-MyBackingImage-",
			expectedBIName:   "",
			expectedDiskUUID: "",
			expectError:      true,
		},
		"ExtractBackingImageAndDiskUUID(...): doesn't have backing image in the lvol name": {
			lvolName:         "-disk-123456-bi-abcdefg",
			expectedBIName:   "",
			expectedDiskUUID: "",
			expectError:      true,
		},
		"ExtractBackingImageAndDiskUUID(...): empty string": {
			lvolName:         "",
			expectedBIName:   "",
			expectedDiskUUID: "",
			expectError:      true,
		},
	}
	for testName, testCase := range testCases {
		c.Logf("testing ExtractBackingImageAndDiskUUID.%v", testName)
		// Call the function being tested
		biName, diskUUID, err := ExtractBackingImageAndDiskUUID(testCase.lvolName)

		if testCase.expectError {
			c.Assert(err, NotNil)
		} else {
			c.Assert(err, IsNil)
			c.Assert(testCase.expectedBIName, Equals, biName)
			c.Assert(testCase.expectedDiskUUID, Equals, diskUUID)
		}
	}
}

func (s *TestSuite) TestSetReplicaAdderInjectsRealFallback(c *C) {
	e := NewEngine("test-engine", "test-volume", types.FrontendEmpty, 1, nil)

	firstMock := &MockReplicaAdder{}
	e.SetReplicaAdder(firstMock)

	firstFallback, ok := firstMock.Real.(*realReplicaAdder)
	c.Assert(ok, Equals, true)
	c.Assert(firstFallback.e, Equals, e)

	secondMock := &MockReplicaAdder{}
	e.SetReplicaAdder(secondMock)

	secondFallback, ok := secondMock.Real.(*realReplicaAdder)
	c.Assert(ok, Equals, true)
	c.Assert(secondFallback.e, Equals, e)
	c.Assert(secondMock.Real == firstMock, Equals, false)
}
</file>

<file path="pkg/spdk/util.go">
package spdk

import (
	"fmt"
	"net"
	"regexp"
	"strconv"
	"strings"
	"time"

	"github.com/avast/retry-go/v4"
	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"

	"github.com/longhorn/go-spdk-helper/pkg/initiator"
	"github.com/longhorn/go-spdk-helper/pkg/jsonrpc"

	commonns "github.com/longhorn/go-common-libs/ns"
	commontypes "github.com/longhorn/go-common-libs/types"
	spdkclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
	spdktypes "github.com/longhorn/go-spdk-helper/pkg/spdk/types"
	helpertypes "github.com/longhorn/go-spdk-helper/pkg/types"
	helperutil "github.com/longhorn/go-spdk-helper/pkg/util"
)

func discoverAndConnectNVMeTarget(srcIP string, srcPort int32, maxRetries int, retryInterval time.Duration) (subsystemNQN, controllerName string, err error) {
	executor, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	if err != nil {
		return "", "", errors.Wrapf(err, "failed to create executor")
	}

	err = retry.Do(
		func() error {
			var e error
			subsystemNQN, e = initiator.DiscoverTarget(srcIP, strconv.Itoa(int(srcPort)), executor)
			if e != nil {
				return errors.Wrapf(e, "discover target %s:%d failed", srcIP, srcPort)
			}

			controllerName, e = initiator.ConnectTarget(srcIP, strconv.Itoa(int(srcPort)), subsystemNQN, executor)
			if e != nil {
				return errors.Wrapf(e, "connect target %s:%d (nqn=%s) failed", srcIP, srcPort, subsystemNQN)
			}

			return nil
		},
		retry.Attempts(uint(maxRetries)),
		retry.Delay(retryInterval),
		retry.DelayType(retry.FixedDelay),
		retry.LastErrorOnly(true),
		retry.OnRetry(func(n uint, err error) {
			logrus.WithError(err).Warnf(
				"Retrying NVMe target connect: addr=%s:%d attempt=%d/%d next_wait=%s",
				srcIP, srcPort, n+1, maxRetries, retryInterval,
			)
		}),
	)

	if err != nil || subsystemNQN == "" || controllerName == "" {
		return "", "", errors.Wrapf(err, "timeout connecting target with address %v:%v", srcIP, srcPort)
	}

	return subsystemNQN, controllerName, nil
}

func exposeSnapshotLvolBdev(spdkClient *spdkclient.Client, lvsName, lvolName, ip string, port int32, executor *commonns.Executor) (subsystemNQN, controllerName string, err error) {
	bdevLvolList, err := spdkClient.BdevLvolGet(spdktypes.GetLvolAlias(lvsName, lvolName), 0)
	if err != nil {
		return "", "", err
	}
	if len(bdevLvolList) == 0 {
		return "", "", errors.Errorf("cannot find lvol bdev %v for backup", lvolName)
	}

	portStr := strconv.Itoa(int(port))
	err = spdkClient.StartExposeBdev(helpertypes.GetNQN(lvolName), bdevLvolList[0].UUID, generateNGUID(lvolName), ip, portStr)
	if err != nil {
		return "", "", errors.Wrapf(err, "failed to expose snapshot lvol bdev %v", lvolName)
	}

	for r := 0; r < maxRetries; r++ {
		subsystemNQN, err = initiator.DiscoverTarget(ip, portStr, executor)
		if err != nil {
			logrus.WithError(err).Errorf("Failed to discover target for snapshot lvol bdev %v", lvolName)
			time.Sleep(retryInterval)
			continue
		}

		controllerName, err = initiator.ConnectTarget(ip, portStr, subsystemNQN, executor)
		if err != nil {
			logrus.WithError(err).Errorf("Failed to connect target for snapshot lvol bdev %v", lvolName)
			time.Sleep(retryInterval)
			continue
		}
		// break when it successfully discover and connect the target
		break
	}
	return subsystemNQN, controllerName, nil
}

func splitHostPort(address string) (string, int32, error) {
	if strings.Contains(address, ":") {
		host, port, err := net.SplitHostPort(address)
		if err != nil {
			return "", 0, errors.Wrapf(err, "failed to split host and port from address %v", address)
		}

		portAsInt := 0
		if port != "" {
			portAsInt, err = strconv.Atoi(port)
			if err != nil {
				return "", 0, errors.Wrapf(err, "failed to parse port %v", port)
			}
		}
		return host, int32(portAsInt), nil
	}

	return address, 0, nil
}

// connectNVMfBdev connects to the NVMe/TCP target, which is exposed by a remote lvol bdev.
// controllerName is typically the lvol name, and address is the IP:port of the NVMe/TCP target.
func connectNVMfBdev(spdkClient *spdkclient.Client, controllerName, address string, ctrlrLossTimeout, fastIOFailTimeoutSec int, maxRetries int, retryInterval time.Duration) (bdevName string, err error) {
	if controllerName == "" || address == "" {
		return "", fmt.Errorf("controllerName or address is empty")
	}

	defer func() {
		if err != nil {
			if _, detachErr := spdkClient.BdevNvmeDetachController(controllerName); detachErr != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(detachErr) {
				logrus.WithError(detachErr).Errorf("Failed to detach NVMe controller %s after failing at attaching it", controllerName)
			}
		}
	}()

	ip, port, err := net.SplitHostPort(address)
	if err != nil {
		return "", err
	}

	// Blindly detach the controller in case of the previous replica connection is not cleaned up correctly
	if _, err := spdkClient.BdevNvmeDetachController(controllerName); err != nil && !jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
		return "", err
	}

	nvmeBdevNameList := []string{}
	err = retry.Do(
		func() error {
			var err error
			nvmeBdevNameList, err = spdkClient.BdevNvmeAttachController(
				controllerName,
				helpertypes.GetNQN(controllerName),
				ip,
				port,
				spdktypes.NvmeTransportTypeTCP,
				spdktypes.NvmeAddressFamilyIPv4,
				int32(ctrlrLossTimeout),
				replicaReconnectDelaySec,
				int32(fastIOFailTimeoutSec),
				replicaMultipath,
			)
			return err
		},
		retry.Attempts(uint(maxRetries)),
		retry.Delay(retryInterval),
		retry.DelayType(retry.FixedDelay),
		retry.LastErrorOnly(true),
		retry.OnRetry(func(n uint, err error) {
			logrus.WithError(err).Warnf(
				"Retrying NVMe bdev attach: controller=%s address=%s attempt=%d/%d next_wait=%s",
				controllerName, address, n+1, maxRetries, retryInterval,
			)
		}),
	)

	if err != nil {
		return "", fmt.Errorf("attach NVMe controller failed after %d attempts: %w", maxRetries, err)
	}

	if len(nvmeBdevNameList) != 1 {
		return "", fmt.Errorf("got zero or multiple results when attaching lvol %s with address %s as a NVMe bdev: %+v", controllerName, address, nvmeBdevNameList)
	}

	return nvmeBdevNameList[0], nil
}

func disconnectNVMfBdev(spdkClient *spdkclient.Client, bdevName string, maxRetries int, retryInterval time.Duration) error {
	if bdevName == "" {
		return nil
	}

	controllerName := helperutil.GetNvmeControllerNameFromNamespaceName(bdevName)

	return retry.Do(
		func() error {
			_, err := spdkClient.BdevNvmeDetachController(controllerName)
			if err != nil {
				if jsonrpc.IsJSONRPCRespErrorNoSuchDevice(err) {
					return nil
				}
				return err
			}
			return nil
		},
		retry.Attempts(uint(maxRetries)),
		retry.Delay(retryInterval),
		retry.DelayType(retry.FixedDelay),
		retry.LastErrorOnly(true),
		retry.OnRetry(func(n uint, err error) {
			logrus.WithError(err).Warnf(
				"Retrying NVMe bdev detach: controller=%s attempt=%d/%d next_wait=%s",
				controllerName, n+1, maxRetries, retryInterval,
			)
		}),
	)
}

func GetSnapXattr(spdkClient *spdkclient.Client, alias, key string) (string, error) {
	value, err := spdkClient.BdevLvolGetXattr(alias, key)
	if err != nil {
		return "", err
	}
	return value, nil
}

func GetLvsNameByUUID(spdkClient *spdkclient.Client, lvsUUID string) (string, error) {
	if lvsUUID == "" {
		return "", fmt.Errorf("empty UUID provided when getting logical volume store name")
	}
	var lvsList []spdktypes.LvstoreInfo
	lvsList, err := spdkClient.BdevLvolGetLvstore("", lvsUUID)
	if err != nil {
		return "", err
	}
	if len(lvsList) != 1 {
		return "", fmt.Errorf("expected exactly one lvstore for UUID %s, but found %d", lvsUUID, len(lvsList))
	}
	return lvsList[0].Name, nil
}

// ExtractBackingImageAndDiskUUID extracts the BackingImageName and DiskUUID from the string pattern "bi-${BackingImageName}-disk-${DiskUUID}"
func ExtractBackingImageAndDiskUUID(lvolName string) (string, string, error) {
	// Define the regular expression pattern
	// This captures the BackingImageName and DiskUUID while allowing for hyphens in both.
	re := regexp.MustCompile(`^bi-([a-zA-Z0-9-]+)-disk-([a-zA-Z0-9-]+)$`)

	// Try to find a match
	matches := re.FindStringSubmatch(lvolName)
	if matches == nil {
		return "", "", fmt.Errorf("lvolName does not match the expected pattern")
	}

	// Extract BackingImageName and DiskUUID from the matches
	backingImageName := matches[1]
	diskUUID := matches[2]

	return backingImageName, diskUUID, nil
}
</file>

<file path="pkg/types/types.go">
package types

import (
	"fmt"
	"strings"

	"github.com/longhorn/types/pkg/generated/spdkrpc"
)

const (
	MetadataDir = "/metadata"
)

type Mode string

const (
	ModeWO  = Mode("WO")
	ModeRW  = Mode("RW")
	ModeERR = Mode("ERR")
)

const (
	FrontendSPDKTCPNvmf     = "spdk-tcp-nvmf"
	FrontendSPDKTCPBlockdev = "spdk-tcp-blockdev"
	FrontendUBLK            = "ublk"
	FrontendEmpty           = ""
)

type InstanceState string

const (
	InstanceStatePending     = "pending"
	InstanceStateStopped     = "stopped"
	InstanceStateRunning     = "running"
	InstanceStateTerminating = "terminating"
	InstanceStateError       = "error"
	InstanceStateSuspended   = "suspended"
)

type InstanceType string

const (
	InstanceTypeReplica        = InstanceType("replica")
	InstanceTypeEngine         = InstanceType("engine")
	InstanceTypeEngineFrontend = InstanceType("engine-frontend")
	InstanceTypeBackingImage   = InstanceType("backingImage")
)

type BackingImageState string

const (
	BackingImageStatePending    = BackingImageState("pending")
	BackingImageStateStarting   = BackingImageState("starting")
	BackingImageStateReady      = BackingImageState("ready")
	BackingImageStateInProgress = BackingImageState("in-progress")
	BackingImageStateFailed     = BackingImageState("failed")
	BackingImageStateUnknown    = BackingImageState("unknown")
)

const (
	BackingImagePortCount = 1
)

const VolumeHead = "volume-head"

const SPDKServicePort = 8504

const (
	DefaultUblkQueueDepth    = 128
	DefaultUblkNumberOfQueue = 1
)

func IsUblkFrontend(frontend string) bool {
	return frontend == FrontendUBLK
}

// IsFrontendSupported returns true if the given frontend type is one of the
// supported frontends (spdk-tcp-blockdev, spdk-tcp-nvmf, ublk, or empty).
func IsFrontendSupported(frontend string) bool {
	switch frontend {
	case FrontendSPDKTCPBlockdev, FrontendSPDKTCPNvmf, FrontendUBLK, FrontendEmpty:
		return true
	default:
		return false
	}
}

func ReplicaModeToGRPCReplicaMode(mode Mode) spdkrpc.ReplicaMode {
	switch mode {
	case ModeWO:
		return spdkrpc.ReplicaMode_WO
	case ModeRW:
		return spdkrpc.ReplicaMode_RW
	case ModeERR:
		return spdkrpc.ReplicaMode_ERR
	}
	return spdkrpc.ReplicaMode_ERR
}

func GRPCReplicaModeToReplicaMode(replicaMode spdkrpc.ReplicaMode) Mode {
	switch replicaMode {
	case spdkrpc.ReplicaMode_WO:
		return ModeWO
	case spdkrpc.ReplicaMode_RW:
		return ModeRW
	case spdkrpc.ReplicaMode_ERR:
		return ModeERR
	}
	return ModeERR
}

const (
	ProgressStateError      = "error"
	ProgressStateComplete   = "complete"
	ProgressStateInProgress = "in_progress"
	ProgressStateStarting   = "starting"

	// SPDKShallowCopyStateNew is the state returned from spdk_tgt. There is no underscore in the string.
	SPDKShallowCopyStateInProgress = "in progress"
	SPDKDeepCopyStateInProgress    = "in progress"
)

// Longhorn defined snapshot attributes
const (
	LonghornBackingImageSnapshotAttrChecksum     = "longhorn_backing_image_checksum"
	LonghornBackingImageSnapshotAttrUUID         = "longhorn_backing_image_uuid"
	LonghornBackingImageSnapshotAttrPrepareState = "longhorn_backing_image_prepare_state"
)

// Backing image related utility functions
const (
	BackingImageTempHeadLvolSuffix = "temp-head"
)

func IsBackingImageSnapLvolName(lvolName string) bool {
	return strings.HasPrefix(lvolName, "bi-") && !strings.HasSuffix(lvolName, BackingImageTempHeadLvolSuffix)
}

func GetBackingImageSnapLvolNameFromTempHeadLvolName(lvolName string) string {
	return strings.TrimSuffix(lvolName, fmt.Sprintf("-%s", BackingImageTempHeadLvolSuffix))
}

func IsBackingImageTempHead(lvolName string) bool {
	return strings.HasSuffix(lvolName, BackingImageTempHeadLvolSuffix)
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
		defer b.Unlock()
	}
	if _, ok := b.subs[sub]; ok {
		close(sub)
		delete(b.subs, sub)
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
	defer b.Unlock()
	for sub := range b.subs {
		b.unsub(sub, false)
	}
	b.running = false
}
</file>

<file path="pkg/util/block_test.go">
package util

import (
	"fmt"
	"os"
	"path/filepath"
	"regexp"
	"testing"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"

	. "gopkg.in/check.v1"
)

func Test(t *testing.T) { TestingT(t) }

type TestSuite struct{}

var _ = Suite(&TestSuite{})

// Mockable variables for dependency injection in tests
var (
	readDir       = os.ReadDir
	readlink      = os.Readlink
	evalSymlinks  = filepath.EvalSymlinks
	sysBlockConst = "/sys/block"
)

func fakeGetDevNameFromBDF(bdf string) (string, error) {
	entries, err := readDir(sysBlockConst)
	if err != nil {
		return "", errors.Wrap(err, "failed to read /sys/block")
	}

	re := regexp.MustCompile(`(?i)` + regexp.QuoteMeta(bdf) + `(/|$)`)

	for _, entry := range entries {
		dev := entry.Name()
		devPath := filepath.Join(sysBlockConst, dev, "device")

		_, err := readlink(devPath)
		if err != nil {
			// Ignore devices without a device symlink (like virtual devices: loop, ramdisk)
			continue
		}

		absPath, err := evalSymlinks(devPath)
		if err != nil {
			continue
		}

		logrus.Infof("Checking %s: resolved path %s", dev, absPath)
		if re.MatchString(absPath) {
			return dev, nil
		}
	}

	return "", fmt.Errorf("device not found for BDF %s", bdf)
}

func (s *TestSuite) TestGetDevNameFromBDF(c *C) {
	fmt.Println("Testing GetDevNameFromBDF with various BDF inputs and mocked sysfs entries")

	type testCase struct {
		name        string
		bdf         string
		expectedDev string
		expectedErr string // error message substring
		setup       func() // optional setup for mocks
	}

	testCases := []testCase{
		{
			name:        "valid BDF matching nvme device",
			bdf:         "0000:02:00.0",
			expectedDev: "nvme0c0n1",
			setup: func() {
				// Mock ReadDir to return real-like entries: loops and nvmess
				readDir = func(path string) ([]os.DirEntry, error) {
					c.Assert(path, Equals, sysBlockConst)
					entries := []os.DirEntry{
						mockDirEntry("loop0"),
						mockDirEntry("nvme0c0n1"),
						mockDirEntry("nvme0n1"),
						mockDirEntry("nvme1c1n1"),
					}
					return entries, nil
				}
				// Mock readlink: fail for loop0, succeed for others
				readlink = func(path string) (string, error) {
					dev := filepath.Base(filepath.Dir(path))
					if dev == "loop0" {
						return "", errors.New("no such file or directory")
					}
					if dev == "nvme0c0n1" {
						return "../../../nvme/nvme0", nil
					}
					if dev == "nvme0n1" {
						return "../../../virtual/nvme-subsystem/nvme-subsys0", nil
					}
					if dev == "nvme1c1n1" {
						return "../../../nvme/nvme1", nil
					}
					return "", errors.New("no link")
				}
				// Mock evalSymlinks: resolve to paths containing or not containing BDF
				evalSymlinks = func(path string) (string, error) {
					dev := filepath.Base(filepath.Dir(path))
					if dev == "nvme0c0n1" {
						return "/sys/devices/pci0000:00/0000:00:08.0/0000:02:00.0/nvme/nvme0/nvme0c0n1/device", nil
					}
					if dev == "nvme0n1" {
						return "/sys/devices/virtual/nvme-subsystem/nvme-subsys0/nvme0n1/device", nil
					}
					if dev == "nvme1c1n1" {
						return "/sys/devices/pci0000:c8/0000:c8:01.0/0000:c9:00.0/nvme/nvme1/nvme1c1n1/device", nil
					}
					return "", errors.New("eval error")
				}
			},
		},
		{
			name:        "valid BDF matching another nvme device",
			bdf:         "0000:c9:00.0",
			expectedDev: "nvme1c1n1",
			setup: func() {
				// Same setup as above
				readDir = func(path string) ([]os.DirEntry, error) {
					c.Assert(path, Equals, sysBlockConst)
					entries := []os.DirEntry{
						mockDirEntry("loop0"),
						mockDirEntry("nvme0c0n1"),
						mockDirEntry("nvme0n1"),
						mockDirEntry("nvme1c1n1"),
					}
					return entries, nil
				}
				readlink = func(path string) (string, error) {
					dev := filepath.Base(filepath.Dir(path))
					if dev == "loop0" {
						return "", errors.New("no such file or directory")
					}
					if dev == "nvme0c0n1" {
						return "../../../nvme/nvme0", nil
					}
					if dev == "nvme0n1" {
						return "../../../virtual/nvme-subsystem/nvme-subsys0", nil
					}
					if dev == "nvme1c1n1" {
						return "../../../nvme/nvme1", nil
					}
					return "", errors.New("no link")
				}
				evalSymlinks = func(path string) (string, error) {
					dev := filepath.Base(filepath.Dir(path))
					if dev == "nvme0c0n1" {
						return "/sys/devices/pci0000:00/0000:00:08.0/0000:02:00.0/nvme/nvme0/nvme0c0n1/device", nil
					}
					if dev == "nvme0n1" {
						return "/sys/devices/virtual/nvme-subsystem/nvme-subsys0/nvme0n1/device", nil
					}
					if dev == "nvme1c1n1" {
						return "/sys/devices/pci0000:c8/0000:c8:01.0/0000:c9:00.0/nvme/nvme1/nvme1c1n1/device", nil
					}
					return "", errors.New("eval error")
				}
			},
		},
		{
			name:        "no devices in sys/block",
			bdf:         "0000:02:00.0",
			expectedErr: "device not found for BDF 0000:02:00.0",
			setup: func() {
				readDir = func(path string) ([]os.DirEntry, error) {
					c.Assert(path, Equals, sysBlockConst)
					return []os.DirEntry{}, nil
				}
			},
		},
		{
			name:        "only virtual devices no match",
			bdf:         "0000:02:00.0",
			expectedErr: "device not found for BDF 0000:02:00.0",
			setup: func() {
				readDir = func(path string) ([]os.DirEntry, error) {
					c.Assert(path, Equals, sysBlockConst)
					return []os.DirEntry{mockDirEntry("loop0"), mockDirEntry("nvme0n1")}, nil
				}
				readlink = func(path string) (string, error) {
					dev := filepath.Base(filepath.Dir(path))
					if dev == "loop0" {
						return "", errors.New("no such file or directory")
					}
					if dev == "nvme0n1" {
						return "../../../virtual/nvme-subsystem/nvme-subsys0", nil
					}
					return "", errors.New("no link")
				}
				evalSymlinks = func(path string) (string, error) {
					dev := filepath.Base(filepath.Dir(path))
					if dev == "nvme0n1" {
						return "/sys/devices/virtual/nvme-subsystem/nvme-subsys0/nvme0n1/device", nil
					}
					return "", errors.New("eval error")
				}
			},
		},
		{
			name:        "devices but no matching BDF",
			bdf:         "ffff:ff:ff.f",
			expectedErr: "device not found for BDF ffff:ff:ff.f",
			setup: func() {
				readDir = func(path string) ([]os.DirEntry, error) {
					c.Assert(path, Equals, sysBlockConst)
					return []os.DirEntry{mockDirEntry("nvme0c0n1")}, nil
				}
				readlink = func(path string) (string, error) {
					return "../../../nvme/nvme0", nil
				}
				evalSymlinks = func(path string) (string, error) {
					return "/sys/devices/pci0000:00/0000:00:08.0/0000:02:00.0/nvme/nvme0/nvme0c0n1/device", nil
				}
			},
		},
		{
			name:        "ReadDir error",
			bdf:         "0000:02:00.0",
			expectedErr: "failed to read /sys/block",
			setup: func() {
				readDir = func(path string) ([]os.DirEntry, error) {
					c.Assert(path, Equals, sysBlockConst)
					return nil, errors.New("permission denied")
				}
			},
		},
		{
			name:        "empty BDF",
			bdf:         "",
			expectedErr: "device not found for BDF ",
			setup: func() {
				readDir = func(path string) ([]os.DirEntry, error) {
					c.Assert(path, Equals, sysBlockConst)
					return []os.DirEntry{}, nil
				}
			},
		},
	}

	for _, tc := range testCases {
		c.Logf("testing GetDevNameFromBDF: %s", tc.name)
		// Reset mocks
		resetMocks()
		if tc.setup != nil {
			tc.setup()
		}

		dev, err := fakeGetDevNameFromBDF(tc.bdf)

		if tc.expectedErr != "" {
			c.Assert(err, NotNil)
			c.Assert(err.Error(), Matches, ".*"+tc.expectedErr+".*")
			c.Assert(dev, Equals, "")
		} else {
			c.Assert(err, IsNil)
			c.Assert(dev, Equals, tc.expectedDev)
		}

		// Reset after test
		resetMocks()
	}
}

// Helper to reset mocks to original functions
func resetMocks() {
	readDir = os.ReadDir
	readlink = os.Readlink
	evalSymlinks = filepath.EvalSymlinks
}

// Mock DirEntry for simplicity
type mockDirEntry string

func (m mockDirEntry) Name() string               { return string(m) }
func (m mockDirEntry) IsDir() bool                { return false }
func (m mockDirEntry) Type() os.FileMode          { return 0 }
func (m mockDirEntry) Info() (os.FileInfo, error) { return nil, nil }
func (m mockDirEntry) Sys() any                   { return nil }
</file>

<file path="pkg/util/block.go">
package util

import (
	"encoding/json"
	"fmt"
	"os"
	"path/filepath"
	"regexp"
	"strconv"
	"strings"

	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"

	"github.com/longhorn/go-spdk-helper/pkg/types"

	commontypes "github.com/longhorn/go-common-libs/types"
	helperutil "github.com/longhorn/go-spdk-helper/pkg/util"
)

func GetDevNameFromBDF(bdf string) (string, error) {
	const sysBlock = "/sys/block"

	entries, err := os.ReadDir(sysBlock)
	if err != nil {
		return "", errors.Wrapf(err, "failed to read %v", sysBlock)
	}

	re := regexp.MustCompile(`/` + regexp.QuoteMeta(bdf) + `(/|$)`)

	for _, entry := range entries {
		dev := entry.Name()
		devPath := filepath.Join(sysBlock, dev, "device")

		absPath, err := filepath.EvalSymlinks(devPath)
		if err != nil {
			// Ignore devices without a device symlink (like virtual devices: loop, ramdisk)
			continue
		}

		logrus.Infof("Checking %s: resolved path %s", dev, absPath)
		if re.MatchString(absPath) {
			return dev, nil
		}
	}

	return "", fmt.Errorf("device not found for BDF %s", bdf)
}

type BlockDevice struct {
	Name       string   `json:"name"`
	Path       string   `json:"path"`
	Subsystems []string `json:"subsystems"`
	Maj        int      `json:"maj"`
	Min        int      `json:"min"`
}

type BlockDevices struct {
	BlockDevices []struct {
		Name       string `json:"name"`
		Path       string `json:"path"`
		MajMin     string `json:"maj:min"`
		Subsystems string `json:"subsystems"`
	} `json:"blockdevices"`
}

func GetBlockDevice(devPath string) (BlockDevice, error) {
	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	if err != nil {
		return BlockDevice{}, errors.Wrap(err, "failed to create executor")
	}

	cmdArgs := []string{"-O", "-J", devPath}
	output, err := ne.Execute(nil, "lsblk", cmdArgs, types.ExecuteTimeout)
	if err != nil {
		return BlockDevice{}, errors.Wrap(err, "failed to get disk subsystems")
	}

	var blockDevices BlockDevices
	err = json.Unmarshal([]byte(output), &blockDevices)
	if err != nil {
		return BlockDevice{}, err
	}

	if len(blockDevices.BlockDevices) == 0 {
		return BlockDevice{}, fmt.Errorf("no blockdevices found")
	}

	bd := blockDevices.BlockDevices[0]
	majMinParts := strings.Split(bd.MajMin, ":")
	if len(majMinParts) != 2 {
		return BlockDevice{}, fmt.Errorf("invalid maj:min format")
	}
	maj, err := strconv.Atoi(majMinParts[0])
	if err != nil {
		return BlockDevice{}, err
	}
	min, err := strconv.Atoi(majMinParts[1])
	if err != nil {
		return BlockDevice{}, err
	}

	subsystems := strings.Split(bd.Subsystems, ":")

	return BlockDevice{
		Name:       bd.Name,
		Path:       bd.Path,
		Subsystems: subsystems,
		Maj:        maj,
		Min:        min,
	}, nil
}
</file>

<file path="pkg/util/http_handler.go">
package util

import (
	"bytes"
	"context"
	"fmt"
	"io"
	"net/http"
	"os"
	"strconv"
	"strings"
	"time"

	"github.com/sirupsen/logrus"
)

const (
	DownloadBufferSize = 1 << 12
	HTTPTimeout        = 10 * time.Second
)

type ProgressUpdater interface {
	UpdateProgress(size int64)
}

type Handler interface {
	GetSizeFromURL(url string) (fileSize int64, err error)
	DownloadFromURL(ctx context.Context, url, outFh *os.File, updater ProgressUpdater) (written int64, err error)
}

type HTTPHandler struct{}

func (h *HTTPHandler) GetSizeFromURL(url string) (size int64, err error) {
	ctx, cancel := context.WithTimeout(context.Background(), HTTPTimeout)
	defer cancel()

	rr, err := http.NewRequestWithContext(ctx, http.MethodHead, url, nil)
	if err != nil {
		return 0, err
	}

	client := NewDownloadHttpClient()
	resp, err := client.Do(rr)
	if err != nil {
		return 0, err
	}
	defer func() {
		if errClose := resp.Body.Close(); errClose != nil {
			logrus.WithError(errClose).Errorf("Failed to close response body for %s", url)
		}
	}()

	if resp.StatusCode != http.StatusOK {
		return 0, fmt.Errorf("expected status code 200 from %s, got %s", url, resp.Status)
	}

	contentLength := resp.Header.Get("Content-Length")
	if contentLength == "" {
		// -1 indicates unknown size
		size = -1
	} else {
		size, err = strconv.ParseInt(contentLength, 10, 64)
		if err != nil {
			return 0, err
		}
	}

	return size, nil
}

func (h *HTTPHandler) DownloadFromURL(ctx context.Context, url string, outFh *os.File, updater ProgressUpdater) (written int64, err error) {
	ctx, cancel := context.WithCancel(ctx)
	defer cancel()

	rr, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
	if err != nil {
		return 0, err
	}

	client := NewDownloadHttpClient()
	resp, err := client.Do(rr)
	if err != nil {
		return 0, err
	}
	defer func() {
		if errClose := resp.Body.Close(); errClose != nil {
			logrus.WithError(errClose).Errorf("Failed to close response body for %s", url)
		}
	}()

	if resp.StatusCode != http.StatusOK {
		return 0, fmt.Errorf("expected status code 200 from %s, got %s", url, resp.Status)
	}

	copied, err := IdleTimeoutCopy(ctx, cancel, resp.Body, outFh, updater, false)
	if err != nil {
		return 0, err
	}

	return copied, nil
}

// IdleTimeoutCopy relies on ctx of the reader/src or a separate timer to interrupt the processing.
func IdleTimeoutCopy(ctx context.Context, cancel context.CancelFunc, src io.ReadCloser, dst io.WriteSeeker, updater ProgressUpdater, writeZero bool) (copied int64, err error) {
	writeSeekCh := make(chan int64, 100)
	defer close(writeSeekCh)

	go func() {
		t := time.NewTimer(HTTPTimeout)
		done := false
		for !done {
			select {
			case <-ctx.Done():
				done = true
			case <-t.C:
				cancel()
				done = true
			case _, writeChOpen := <-writeSeekCh:
				if !writeChOpen {
					done = true
					break
				}
				if !t.Stop() {
					<-t.C
				}
				t.Reset(HTTPTimeout)
			}
		}

		// Still need to make sure to clean up the signals in writeSeekCh
		// so that they won't block the below sender.
		for writeChOpen := true; writeChOpen; {
			_, writeChOpen = <-writeSeekCh
		}
	}()

	var nr, nw int
	var nws int64
	var rErr, handleErr error
	buf := make([]byte, DownloadBufferSize)
	zeroByteArray := make([]byte, DownloadBufferSize)
	for rErr == nil && err == nil {
		select {
		case <-ctx.Done():
			err = fmt.Errorf("context cancelled during the copy")
		default:
			// Read will error out once the context is cancelled.
			nr, rErr = src.Read(buf)
			if nr > 0 {
				// Skip writing zero data
				if !writeZero && bytes.Equal(buf[0:nr], zeroByteArray[0:nr]) {
					_, handleErr = dst.Seek(int64(nr), io.SeekCurrent)
					nws = int64(nr)
				} else {
					nw, handleErr = dst.Write(buf[0:nr])
					nws = int64(nw)
				}
				if handleErr != nil {
					err = handleErr
					break
				}
				writeSeekCh <- nws
				copied += nws
				updater.UpdateProgress(nws)
			}
			if rErr != nil {
				if rErr != io.EOF {
					err = rErr
				}
				break // nolint: staticcheck
			}
		}
	}

	return copied, err
}

func removeReferer(req *http.Request) {
	for k := range req.Header {
		if strings.ToLower(k) == "referer" {
			delete(req.Header, k)
		}
	}
}

func NewDownloadHttpClient() http.Client {
	return http.Client{
		CheckRedirect: func(req *http.Request, via []*http.Request) error {
			// Remove the "Referer" header to enable downloads of files
			// that are delivered via CDN and therefore may be redirected
			// several times. This is the same behaviour of curl or wget
			// in their default configuration.
			removeReferer(req)
			return nil
		},
	}
}
</file>

<file path="pkg/util/util.go">
package util

import (
	"crypto/sha512"
	"encoding/hex"
	"fmt"
	"io"
	"os"
	"os/exec"
	"regexp"
	"strings"
	"syscall"
	"time"

	"github.com/cockroachdb/errors"
	"github.com/google/uuid"
	"github.com/sirupsen/logrus"
	"go.uber.org/multierr"

	"k8s.io/apimachinery/pkg/util/validation"

	"github.com/longhorn/go-common-libs/proc"
)

// Param represents a named string parameter.
type Param struct {
	Name  string
	Value string
}

// VerifyParams checks that none of the provided Params are empty.
// It returns an error listing all missing parameter names, or nil if all are present.
func VerifyParams(params ...Param) error {
	var missing []string
	for _, p := range params {
		if strings.TrimSpace(p.Value) == "" {
			missing = append(missing, p.Name)
		}
	}
	if len(missing) > 0 {
		return fmt.Errorf("missing required parameter(s): %s", strings.Join(missing, ", "))
	}
	return nil
}

func RoundUp(num, base uint64) uint64 {
	if num <= 0 {
		return base
	}
	r := num % base
	if r == 0 {
		return num
	}
	return num - r + base
}

const (
	EngineRandomIDLenth = 8
	EngineSuffix        = "-e"
)

func GetVolumeNameFromEngineName(engineName string) string {
	reg := regexp.MustCompile(fmt.Sprintf(`([^"]*)%s-[A-Za-z0-9]{%d,%d}$`, EngineSuffix, EngineRandomIDLenth, EngineRandomIDLenth))
	return reg.ReplaceAllString(engineName, "${1}")
}

func BytesToMiB(bytes uint64) uint64 {
	return bytes / 1024 / 1024
}

func RemovePrefix(path, prefix string) string {
	if strings.HasPrefix(path, prefix) {
		return strings.TrimPrefix(path, prefix)
	}
	return path
}

func UUID() string {
	return uuid.New().String()
}

func IsSPDKTargetProcessRunning() (bool, error) {
	cmd := exec.Command("pgrep", "-f", "spdk_tgt")
	if _, err := cmd.Output(); err != nil {
		if exitErr, ok := err.(*exec.ExitError); ok {
			status, ok := exitErr.Sys().(syscall.WaitStatus)
			if ok {
				exitCode := status.ExitStatus()
				if exitCode == 1 {
					return false, nil
				}
			}
		}
		return false, errors.Wrap(err, "failed to check spdk_tgt process")
	}
	return true, nil
}

func StartSPDKTgtDaemon() error {
	cmd := exec.Command("spdk_tgt")

	cmd.SysProcAttr = &syscall.SysProcAttr{
		Setsid: true,
	}

	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	err := cmd.Start()
	if err != nil {
		return fmt.Errorf("failed to start spdk_tgt daemon: %w", err)
	}

	return nil
}

func ForceStopSPDKTgtDaemon(timeout time.Duration) error {
	return stopSPDKTgtDaemon(timeout, syscall.SIGKILL)
}

func StopSPDKTgtDaemon(timeout time.Duration) error {
	return stopSPDKTgtDaemon(timeout, syscall.SIGTERM)
}

func stopSPDKTgtDaemon(timeout time.Duration, signal syscall.Signal) error {
	processes, err := proc.FindProcessByCmdline("spdk_tgt")
	if err != nil {
		return errors.Wrap(err, "failed to find spdk_tgt")
	}

	var errs error
	for _, process := range processes {
		logrus.Infof("Sending signal %v to spdk_tgt %v", signal, process.Pid)
		if err := process.Signal(signal); err != nil {
			errs = multierr.Append(errs, errors.Wrapf(err, "failed to send signal %v to spdk_tgt %v", signal, process.Pid))
		} else {
			done := make(chan error, 1)
			go func() {
				_, err := process.Wait()
				done <- err
				close(done)
			}()

			select {
			case <-time.After(timeout):
				logrus.Warnf("spdk_tgt %v failed to exit in time, sending signal %v", process.Pid, signal)
				err = process.Signal(signal)
				if err != nil {
					errs = multierr.Append(errs, errors.Wrapf(err, "failed to send signal %v to spdk_tgt %v", signal, process.Pid))
				}
			case err := <-done:
				if err != nil {
					errs = multierr.Append(errs, errors.Wrapf(err, "spdk_tgt %v exited with error", process.Pid))
				} else {
					logrus.Infof("spdk_tgt %v exited successfully", process.Pid)
				}
			}
		}
	}

	return errs
}

func GetFileChunkChecksum(filePath string, start, size int64) (string, error) {
	f, err := os.Open(filePath)
	if err != nil {
		return "", err
	}
	defer func() {
		if errClose := f.Close(); errClose != nil {
			logrus.WithError(errClose).Errorf("Failed to close file %s", filePath)
		}
	}()

	if _, err = f.Seek(start, 0); err != nil {
		return "", err
	}

	h := sha512.New()
	if _, err := io.CopyN(h, f, size); err != nil {
		return "", err
	}

	return hex.EncodeToString(h.Sum(nil)), nil
}
func ParseLabels(labels []string) (map[string]string, error) {
	result := map[string]string{}
	for _, label := range labels {
		kv := strings.SplitN(label, "=", 2)
		if len(kv) != 2 {
			return nil, fmt.Errorf("invalid label not in <key>=<value> format %v", label)
		}
		key := kv[0]
		value := kv[1]
		if errList := validation.IsQualifiedName(key); len(errList) > 0 {
			return nil, fmt.Errorf("invalid key %v for label: %v", key, errList[0])
		}
		// We don't need to validate the Label value since we're allowing for any form of data to be stored, similar
		// to Kubernetes Annotations. Of course, we should make sure it isn't empty.
		if value == "" {
			return nil, fmt.Errorf("invalid empty value for label with key %v", key)
		}
		result[key] = value
	}
	return result, nil
}

func Now() string {
	return time.Now().UTC().Format(time.RFC3339)
}

func UnescapeURL(url string) string {
	// Deal with escape in url inputted from bash
	result := strings.Replace(url, "\\u0026", "&", 1)
	result = strings.Replace(result, "u0026", "&", 1)
	result = strings.TrimLeft(result, "\"'")
	result = strings.TrimRight(result, "\"'")
	return result
}

func CombineErrors(errorList ...error) (retErr error) {
	for _, err := range errorList {
		if err != nil {
			if retErr != nil {
				retErr = fmt.Errorf("%v, %v", retErr, err)
			} else {
				retErr = err
			}
		}
	}
	return retErr
}

func Min(x, y uint64) uint64 {
	if x < y {
		return x
	}
	return y
}

type ReplicaError struct {
	Address string
	Message string
}

func NewReplicaError(address string, err error) ReplicaError {
	return ReplicaError{
		Address: address,
		Message: err.Error(),
	}
}

func (e ReplicaError) Error() string {
	return fmt.Sprintf("%v: %v", e.Address, e.Message)
}

type TaskError struct {
	ReplicaErrors []ReplicaError
}

func NewTaskError(res ...ReplicaError) *TaskError {
	return &TaskError{
		ReplicaErrors: append([]ReplicaError{}, res...),
	}
}

func (t *TaskError) Error() string {
	var errs []string
	for _, re := range t.ReplicaErrors {
		errs = append(errs, re.Error())
	}

	if errs == nil {
		return "Unknown"
	}
	if len(errs) == 1 {
		return errs[0]
	}
	return strings.Join(errs, "; ")
}

func (t *TaskError) Append(re ReplicaError) {
	t.ReplicaErrors = append(t.ReplicaErrors, re)
}

func (t *TaskError) HasError() bool {
	return len(t.ReplicaErrors) != 0
}

func GetFileChecksum(filePath string) (string, error) {
	f, err := os.Open(filePath)
	if err != nil {
		return "", err
	}
	defer func() {
		if errClose := f.Close(); errClose != nil {
			logrus.WithError(errClose).Errorf("Failed to close file %s", filePath)
		}
	}()

	h := sha512.New()
	if _, err := io.Copy(h, f); err != nil {
		return "", err
	}

	return hex.EncodeToString(h.Sum(nil)), nil
}
</file>

<file path="pkg/spdk_test.go">
package pkg

import (
	"context"
	"fmt"
	"net"
	"os"
	"path/filepath"
	"reflect"
	"strconv"
	"strings"
	"sync"
	"testing"
	"time"

	"github.com/avast/retry-go/v4"
	"github.com/cockroachdb/errors"
	"github.com/sirupsen/logrus"
	"google.golang.org/grpc"
	"google.golang.org/grpc/keepalive"
	"google.golang.org/grpc/reflection"

	grpccodes "google.golang.org/grpc/codes"
	grpcstatus "google.golang.org/grpc/status"

	"github.com/longhorn/types/pkg/generated/spdkrpc"

	commonnet "github.com/longhorn/go-common-libs/net"
	commonns "github.com/longhorn/go-common-libs/ns"
	commontypes "github.com/longhorn/go-common-libs/types"
	helperinitiator "github.com/longhorn/go-spdk-helper/pkg/initiator"
	helperclient "github.com/longhorn/go-spdk-helper/pkg/spdk/client"
	helpertypes "github.com/longhorn/go-spdk-helper/pkg/types"
	helperutil "github.com/longhorn/go-spdk-helper/pkg/util"

	"github.com/longhorn/longhorn-spdk-engine/pkg/api"
	"github.com/longhorn/longhorn-spdk-engine/pkg/client"
	"github.com/longhorn/longhorn-spdk-engine/pkg/types"
	"github.com/longhorn/longhorn-spdk-engine/pkg/util"

	server "github.com/longhorn/longhorn-spdk-engine/pkg/spdk"

	. "gopkg.in/check.v1"
)

var (
	defaultTestDiskName = "test-disk"
	defaultTestDiskPath = filepath.Join("/tmp", defaultTestDiskName)

	defaultTestBlockSize          = 4096
	defaultTestDiskSize           = uint64(20480 * helpertypes.MiB)
	defaultTestLvolSizeInMiB      = uint64(500)
	defaultTestLvolSize           = defaultTestLvolSizeInMiB * helpertypes.MiB
	defaultTestLargeLvolSizeInMiB = uint64(2000)
	defaultTestLargeLvolSize      = defaultTestLargeLvolSizeInMiB * helpertypes.MiB

	defaultTestBackingImageName        = "parrot"
	defaultTestBackingImageUUID        = "12345"
	defaultTestBackingImageChecksum    = "304f3ed30ca6878e9056ee6f1b02b328239f0d0c2c1272840998212f9734b196371560b3b939037e4f4c2884ce457c2cbc9f0621f4f5d1ca983983c8cdf8cd9a"
	defaultTestBackingImageDownloadURL = "https://longhorn-backing-image.s3-us-west-1.amazonaws.com/parrot.raw"
	defaultTestBackingImageSizeInMiB   = uint64(32)
	defaultTestBackingImageSize        = defaultTestBackingImageSizeInMiB * helpertypes.MiB

	defaultTestStartPort        = int32(20000)
	defaultTestEndPort          = int32(30000)
	defaultTestReplicaPortCount = int32(5)

	defaultTestFastSync = true

	// Use a larger timeout for the test to avoid timeout issues while enabling debugger such as valgrind.
	defaultTestExecuteTimeout = 120 * time.Second

	defaultTestRebuildingWaitInterval   = 3 * time.Second
	defaultTestRebuildingWaitCount      = 60
	defaultTestSnapChecksumWaitInterval = 1 * time.Second
	defaultTestSnapChecksumWaitCount    = 60

	maxBackingImageGetRetries = 300

	checkReplicaSnapshotsMaxRetries   = 6
	checkReplicaSnapshotsWaitInterval = 10 * time.Second

	SPDKTGTBinary = "spdk_tgt"
)

const (
	spdkTargetProbeTimeout      = 3 * time.Second
	spdkTargetStopGracePeriod   = 10 * time.Second
	spdkTargetStartupProbeDelay = 1 * time.Second
	spdkTargetStopPollInterval  = 200 * time.Millisecond
)

func Test(t *testing.T) { TestingT(t) }

type TestSuite struct{}

var _ = Suite(&TestSuite{})
var runtimeMonitoringTestMu sync.Mutex

func getVolumeName() string {
	return fmt.Sprintf("test-vol-%s", time.Now().Format("20060102150405"))
}

func startTarget(spdkDir string, args []string, execute func(envs []string, binary string, args []string, timeout time.Duration) (string, error)) (err error) {
	argsInStr := ""
	for _, arg := range args {
		argsInStr = fmt.Sprintf("%s %s", argsInStr, arg)
	}

	tgtOpts := []string{
		"-c",
		fmt.Sprintf("%s %s", filepath.Join(spdkDir, SPDKTGTBinary), argsInStr),
	}

	// Use a larger timeout for the test to avoid timeout issues while enabling debugger such as valgrind.
	_, err = execute(nil, "sh", tgtOpts, 180*time.Minute)
	return err
}

func probeSPDKTargetReady(timeout time.Duration) bool {
	ctx, cancel := context.WithTimeout(context.Background(), timeout)
	defer cancel()

	spdkCli, err := helperclient.NewClient(ctx)
	if err != nil {
		return false
	}
	defer func() {
		if err := spdkCli.Close(); err != nil {
			logrus.WithError(err).Warn("Failed to close SPDK target probe client")
		}
	}()

	return true
}

func waitForSPDKTargetDaemonStoppedWithTimeout(timeout, pollInterval time.Duration) error {
	deadline := time.Now().Add(timeout)
	for {
		running, err := util.IsSPDKTargetProcessRunning()
		if err != nil {
			return err
		}
		if !running {
			return nil
		}
		if time.Now().After(deadline) {
			return fmt.Errorf("timed out waiting for spdk_tgt to stop")
		}
		time.Sleep(pollInterval)
	}
}

func ensureSPDKTargetDaemonStopped(timeout time.Duration) error {
	running, err := util.IsSPDKTargetProcessRunning()
	if err != nil {
		return err
	}
	if !running {
		return nil
	}

	if err := util.ForceStopSPDKTgtDaemon(timeout); err != nil &&
		!strings.Contains(err.Error(), "process with cmdline spdk_tgt is not found") {
		return err
	}

	return waitForSPDKTargetDaemonStoppedWithTimeout(timeout, spdkTargetStopPollInterval)
}

func LaunchTestSPDKTargetDaemon(c *C, execute func(envs []string, name string, args []string, timeout time.Duration) (string, error)) {
	err := ensureSPDKTargetDaemonStopped(spdkTargetStopGracePeriod)
	c.Assert(err, IsNil)

	targetReady := false
	go func() {
		logrus.Info("Starting SPDK target daemon")
		err := startTarget("", []string{"--logflag all", "2>&1 | tee /tmp/spdk_tgt.log"}, execute)
		c.Assert(err, IsNil)
	}()

	for cnt := 0; cnt < 300; cnt++ {
		if probeSPDKTargetReady(spdkTargetProbeTimeout) {
			targetReady = true
			break
		}
		time.Sleep(spdkTargetStartupProbeDelay)
	}

	c.Assert(targetReady, Equals, true)
}

// launchTestSPDKGRPCServer launches the SPDK gRPC server and returns the server instance.
// This helper allows tests to access the server instance for advanced testing scenarios (e.g. error injection).
func launchTestSPDKGRPCServer(ctx context.Context, c *C, ip string, execute func(envs []string, name string, args []string, timeout time.Duration) (string, error), wg *sync.WaitGroup) *server.Server {

	LaunchTestSPDKTargetDaemon(c, execute)
	srv, err := server.NewServer(ctx, defaultTestStartPort, defaultTestEndPort)
	c.Assert(err, IsNil)

	spdkGRPCListener, err := net.Listen("tcp", net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
	c.Assert(err, IsNil)
	spdkGRPCServer := grpc.NewServer(grpc.KeepaliveEnforcementPolicy(keepalive.EnforcementPolicy{
		MinTime:             10 * time.Second,
		PermitWithoutStream: true,
	}))

	wg.Add(1)
	go func() {
		defer wg.Done()

		<-ctx.Done()
		spdkGRPCServer.Stop()
		logrus.Info("Stopping SPDK gRPC server")
		// TODO: The error "no such child process" will be emitted when the process is already stopped.
		// Need to improve the error handling, but we can ignore it for now.
		if err := util.ForceStopSPDKTgtDaemon(120 * time.Second); err != nil &&
			!strings.Contains(err.Error(), "process with cmdline spdk_tgt is not found") {
			logrus.WithError(err).Warn("Failed to force stop SPDK target daemon")
		}
		if err := waitForSPDKTargetDaemonStoppedWithTimeout(spdkTargetStopGracePeriod, spdkTargetStopPollInterval); err != nil {
			logrus.WithError(err).Warn("SPDK target daemon is still running after stop")
		}
	}()
	spdkrpc.RegisterSPDKServiceServer(spdkGRPCServer, srv)
	reflection.Register(spdkGRPCServer)
	go func() {
		if err := spdkGRPCServer.Serve(spdkGRPCListener); err != nil {
			logrus.WithError(err).Error("Stopping SPDK gRPC server")
		}
	}()
	return srv
}

// LaunchTestSPDKGRPCServer launches the SPDK gRPC server.
// It wraps launchTestSPDKGRPCServer but discards the returned server instance to maintain backward compatibility.
func LaunchTestSPDKGRPCServer(ctx context.Context, c *C, ip string, execute func(envs []string, name string, args []string, timeout time.Duration) (string, error), wg *sync.WaitGroup) {
	launchTestSPDKGRPCServer(ctx, c, ip, execute, wg)
}

func PrepareDiskFile(c *C) string {
	err := os.RemoveAll(defaultTestDiskPath)
	c.Assert(err, IsNil)

	f, err := os.Create(defaultTestDiskPath)
	c.Assert(err, IsNil)
	err = f.Close()
	c.Assert(err, IsNil)

	err = os.Truncate(defaultTestDiskPath, int64(defaultTestDiskSize))
	c.Assert(err, IsNil)

	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	c.Assert(err, IsNil)
	output, err := ne.Execute(nil, "losetup", []string{"-f"}, defaultTestExecuteTimeout)
	c.Assert(err, IsNil)

	loopDevicePath := strings.TrimSpace(output)
	c.Assert(loopDevicePath, Not(Equals), "")

	_, err = ne.Execute(nil, "losetup", []string{loopDevicePath, defaultTestDiskPath}, defaultTestExecuteTimeout)
	c.Assert(err, IsNil)

	return loopDevicePath
}

func CleanupDiskFile(c *C, loopDevicePath string) {
	defer func() {
		err := os.RemoveAll(defaultTestDiskPath)
		c.Assert(err, IsNil)
	}()

	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	c.Assert(err, IsNil)
	_, err = ne.Execute(nil, "losetup", []string{"-d", loopDevicePath}, time.Second)
	c.Assert(err, IsNil)
}

func waitForDiskReady(ctx context.Context, spdkCli *client.SPDKClient, loopDevicePath, diskDriverName string) (*spdkrpc.Disk, error) {
	opts := []retry.Option{
		retry.Context(ctx),
		retry.Attempts(60),
		retry.DelayType(retry.FixedDelay),
		retry.LastErrorOnly(true),
		retry.Delay(1 * time.Second),
	}

	var readyDisk *spdkrpc.Disk

	err := retry.Do(func() error {
		logrus.Info("Checking if the disk is in 'ready' state")
		disk, err := spdkCli.DiskGet(defaultTestDiskName, loopDevicePath, diskDriverName)
		if err != nil {
			logrus.WithError(err).Warnf("Failed to get disk %s", defaultTestDiskName)
			return err
		}
		logrus.Infof("Got disk %s in state %s", disk.Name, disk.State)
		if disk.State != string(server.DiskStateReady) {
			return fmt.Errorf("disk %s is in state %s", disk.Name, disk.State)
		}
		logrus.Infof("Disk %s is in %v state", disk.Name, disk.State)
		readyDisk = disk
		return nil
	}, opts...)
	if err != nil {
		return nil, fmt.Errorf("failed to wait for disk %s to be in 'ready' state: %w", defaultTestDiskName, err)
	}

	return readyDisk, nil
}

// formatBlockDevice formats block device to check if the engine frontend is working
func formatBlockDevice(endpoint string, fsType string) error {
	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	if err != nil {
		return err
	}
	if _, err := ne.Execute(nil, "mkfs", []string{"-t", fsType, endpoint}, defaultTestExecuteTimeout); err != nil {
		return err
	}
	return nil
}

// verifyNVMfInitiatorConnected verifies the NVMe/TCP initiator is connected.
func verifyNVMfInitiatorConnected(engineFrontend *api.EngineFrontend, nqn string) error {
	ip := engineFrontend.TargetIP
	port := strconv.Itoa(int(engineFrontend.TargetPort))

	nvmeTCPInfo := &helperinitiator.NVMeTCPInfo{
		SubsystemNQN:       nqn,
		TransportAddress:   ip,
		TransportServiceID: port,
	}

	// We use empty host proc path here since we are running checking inside the container (same as the host)
	// and we don't want to lock the file for the test.
	initiator, err := helperinitiator.NewInitiator(engineFrontend.VolumeName, helperinitiator.HostProc, nvmeTCPInfo, nil)
	if err != nil {
		return err
	}

	if err := initiator.WaitForNVMeTCPConnect(60, time.Second); err != nil {
		return err
	}

	return nil
}

func (s *TestSuite) TestReplicaCreateWithStateChecks(c *C) {
	fmt.Println("Testing SPDK replica creation with state checks")

	diskDriverName := "aio"

	ip, err := commonnet.GetAnyExternalIP()
	c.Assert(err, IsNil)
	err = os.Setenv(commonnet.EnvPodIP, ip)
	c.Assert(err, IsNil)

	ctx, cancel := context.WithCancel(context.Background())
	var spdkWg sync.WaitGroup
	defer func() {
		cancel()
		spdkWg.Wait()
	}()

	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	c.Assert(err, IsNil)
	LaunchTestSPDKGRPCServer(ctx, c, ip, ne.Execute, &spdkWg)

	loopDevicePath := PrepareDiskFile(c)
	defer func() {
		CleanupDiskFile(c, loopDevicePath)
	}()

	spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
	c.Assert(err, IsNil)
	defer func() {
		if errClose := spdkCli.Close(); errClose != nil {
			logrus.WithError(errClose).Error("Failed to close SPDK client")
		}
	}()

	disk, err := spdkCli.DiskCreate(defaultTestDiskName, "", loopDevicePath, diskDriverName, int64(defaultTestBlockSize))
	c.Assert(err, IsNil)
	c.Assert(disk, NotNil)

	disk, err = waitForDiskReady(ctx, spdkCli, loopDevicePath, diskDriverName)
	c.Assert(err, IsNil)
	c.Assert(disk.Path, Equals, loopDevicePath)
	c.Assert(disk.Uuid, Not(Equals), "")

	defer func() {
		err := spdkCli.DiskDelete(defaultTestDiskName, disk.Uuid, disk.Path, diskDriverName)
		c.Assert(err, IsNil)
	}()

	replicaName := "test-replica-state"
	defer func() {
		_ = spdkCli.ReplicaDelete(replicaName, true)
	}()

	// Pending -> first create should succeed
	replica, err := spdkCli.ReplicaCreate(replicaName, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
	c.Assert(err, IsNil)
	c.Assert(replica.State, Equals, types.InstanceStateRunning)

	// Running -> create again should return AlreadyExists
	_, err = spdkCli.ReplicaCreate(replicaName, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
	c.Assert(err, NotNil)
	rootErr := errors.UnwrapAll(err)
	st, ok := grpcstatus.FromError(rootErr)
	c.Assert(ok, Equals, true)
	c.Assert(st.Code(), Equals, grpccodes.AlreadyExists)
	c.Assert(strings.Contains(st.Message(), "already exists and running"), Equals, true)

	// Stopped -> delete without cleanup then create again should succeed
	err = spdkCli.ReplicaDelete(replicaName, false)
	c.Assert(err, IsNil)

	replica, err = spdkCli.ReplicaGet(replicaName)
	c.Assert(err, IsNil)
	c.Assert(replica.State, Equals, types.InstanceStateStopped)

	replica, err = spdkCli.ReplicaCreate(replicaName, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
	c.Assert(err, IsNil)
	c.Assert(replica.State, Equals, types.InstanceStateRunning)
}

func (s *TestSuite) TestReplicaDeletePendingWithoutCleanupIsNoop(c *C) {
	fmt.Println("Testing SPDK pending replica delete without cleanup is no-op")

	r := server.NewReplica(context.Background(), "test-replica-pending", "test-lvs", "test-lvs-uuid", 0, true, make(chan interface{}, 1))
	c.Assert(r, NotNil)
	c.Assert(string(r.State), Equals, types.InstanceStatePending)

	err := r.Delete(nil, false, nil)
	c.Assert(err, IsNil)
	c.Assert(string(r.State), Equals, types.InstanceStatePending)

	select {
	case <-r.UpdateCh:
		c.Fatalf("unexpected update signal for pending replica delete without cleanup")
	default:
	}
}

func (s *TestSuite) TestSPDKEngineFrontendCreateWithoutEngine(c *C) {
	fmt.Println("Testing SPDK basic operations: engine frontend create without engine")

	diskDriverName := "aio"

	ip, err := commonnet.GetAnyExternalIP()
	c.Assert(err, IsNil)
	err = os.Setenv(commonnet.EnvPodIP, ip)
	c.Assert(err, IsNil)

	ctx, cancel := context.WithCancel(context.Background())
	var spdkWg sync.WaitGroup

	defer func() {
		cancel()
		spdkWg.Wait()
	}()

	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	c.Assert(err, IsNil)
	LaunchTestSPDKGRPCServer(ctx, c, ip, ne.Execute, &spdkWg)

	loopDevicePath := PrepareDiskFile(c)
	defer func() {
		CleanupDiskFile(c, loopDevicePath)
	}()

	spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
	c.Assert(err, IsNil)
	defer func() {
		if errClose := spdkCli.Close(); errClose != nil {
			logrus.WithError(errClose).Error("Failed to close SPDK client")
		}
	}()

	disk, err := spdkCli.DiskCreate(defaultTestDiskName, "", loopDevicePath, diskDriverName, int64(defaultTestBlockSize))
	c.Assert(err, IsNil)
	c.Assert(disk, NotNil)

	disk, err = waitForDiskReady(ctx, spdkCli, loopDevicePath, diskDriverName)
	c.Assert(err, IsNil)
	c.Assert(disk.Path, Equals, loopDevicePath)
	c.Assert(disk.Uuid, Not(Equals), "")

	defer func() {
		err := spdkCli.DiskDelete(defaultTestDiskName, disk.Uuid, disk.Path, diskDriverName)
		c.Assert(err, IsNil)

		disk, err = spdkCli.DiskGet(defaultTestDiskName, disk.Path, diskDriverName)
		c.Assert(err, NotNil)
		c.Assert(disk, IsNil)
	}()

	volumeName := getVolumeName()
	engineName := fmt.Sprintf("%s-e", volumeName)
	engineFrontendName := fmt.Sprintf("%s-ef", volumeName)

	defer func() {
		err = spdkCli.EngineFrontendDelete(engineFrontendName)
		c.Assert(err, IsNil)
	}()

	// Create engine frontend (NVMe/TCP initiator)
	engineFrontend, err := spdkCli.EngineFrontendCreate(engineFrontendName, volumeName, engineName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize,
		net.JoinHostPort(ip, strconv.Itoa(30000)), 0, 0)
	c.Assert(err, IsNil)
	c.Assert(engineFrontend.State, Equals, types.InstanceStateError)
	c.Assert(engineFrontend.ErrorMsg, Not(Equals), "")
	c.Assert(engineFrontend.TargetIP, Equals, "")
	c.Assert(engineFrontend.TargetPort, Equals, int32(0))
}

func (s *TestSuite) TestSPDKEngineAndEngineFrontendCreateAndDeleteWithDifferentFrontends(c *C) {
	fmt.Println("Testing SPDK basic operations: engine/enginefrontend create and delete with different frontends")

	tests := []struct {
		name                 string
		frontend             string
		expectEngineEndpoint bool
		verifyFrontend       func(c *C, volumeName, engineName string, engine *api.Engine, engineFrontend *api.EngineFrontend)
	}{
		{
			name:                 "FrontendSPDKTCPBlockdev",
			frontend:             types.FrontendSPDKTCPBlockdev,
			expectEngineEndpoint: true,
			verifyFrontend: func(c *C, volumeName, _ string, _ *api.Engine, engineFrontend *api.EngineFrontend) {
				endpoint := helperutil.GetLonghornDevicePath(volumeName)
				c.Assert(engineFrontend.Endpoint, Equals, endpoint)
				err := formatBlockDevice(endpoint, "ext4")
				c.Assert(err, IsNil)
			},
		},
		{
			name:                 "FrontendSPDKTCPNvmf",
			frontend:             types.FrontendSPDKTCPNvmf,
			expectEngineEndpoint: true,
			verifyFrontend: func(c *C, _ string, engineName string, engine *api.Engine, engineFrontend *api.EngineFrontend) {
				nqn := helpertypes.GetNQN(engineName)
				endpoint := server.GetNvmfEndpoint(nqn, engine.IP, engine.Port)
				c.Assert(engineFrontend.Endpoint, Equals, endpoint)

				ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
				c.Assert(err, IsNil)
				_, err = helperinitiator.ConnectTarget(engine.IP, strconv.Itoa(int(engine.Port)), nqn, ne)
				c.Assert(err, IsNil)

				err = verifyNVMfInitiatorConnected(engineFrontend, nqn)
				c.Assert(err, IsNil)

				time.Sleep(5 * time.Second)

				err = helperinitiator.DisconnectTarget(nqn, ne)
				c.Assert(err, IsNil)
			},
		},
		{
			name:                 "FrontendUBLK",
			frontend:             types.FrontendUBLK,
			expectEngineEndpoint: false,
			verifyFrontend: func(c *C, volumeName, _ string, _ *api.Engine, engineFrontend *api.EngineFrontend) {
				endpoint := helperutil.GetLonghornDevicePath(volumeName)
				c.Assert(engineFrontend.Endpoint, Equals, endpoint)

				err := formatBlockDevice(endpoint, "ext4")
				c.Assert(err, IsNil)
				c.Assert(engineFrontend.UblkID, Not(Equals), int32(helperinitiator.UnInitializedUblkId))
			},
		},
	}

	assertEngineEndpoint := func(c *C, ip string, port int32, expectSet bool) {
		if expectSet {
			c.Assert(ip, Not(Equals), "")
			c.Assert(port, Not(Equals), int32(0))
			return
		}
		c.Assert(ip, Equals, "")
		c.Assert(port, Equals, int32(0))
	}

	for _, test := range tests {
		test := test
		fmt.Println("Testing frontend case:", test.name)

		func() {
			diskDriverName := "aio"

			ip, err := commonnet.GetAnyExternalIP()
			c.Assert(err, IsNil)
			err = os.Setenv(commonnet.EnvPodIP, ip)
			c.Assert(err, IsNil)

			ctx, cancel := context.WithCancel(context.Background())
			var spdkWg sync.WaitGroup
			defer func() {
				cancel()
				spdkWg.Wait()
			}()

			ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
			c.Assert(err, IsNil)
			LaunchTestSPDKGRPCServer(ctx, c, ip, ne.Execute, &spdkWg)

			loopDevicePath := PrepareDiskFile(c)
			defer func() {
				CleanupDiskFile(c, loopDevicePath)
			}()

			spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
			c.Assert(err, IsNil)
			defer func() {
				if errClose := spdkCli.Close(); errClose != nil {
					logrus.WithError(errClose).Error("Failed to close SPDK client")
				}
			}()

			disk, err := spdkCli.DiskCreate(defaultTestDiskName, "", loopDevicePath, diskDriverName, int64(defaultTestBlockSize))
			c.Assert(err, IsNil)
			c.Assert(disk, NotNil)

			disk, err = waitForDiskReady(ctx, spdkCli, loopDevicePath, diskDriverName)
			c.Assert(err, IsNil)
			c.Assert(disk.Path, Equals, loopDevicePath)
			c.Assert(disk.Uuid, Not(Equals), "")

			defer func() {
				err := spdkCli.DiskDelete(defaultTestDiskName, disk.Uuid, disk.Path, diskDriverName)
				c.Assert(err, IsNil)

				disk, err = spdkCli.DiskGet(defaultTestDiskName, disk.Path, diskDriverName)
				c.Assert(err, NotNil)
				c.Assert(disk, IsNil)
			}()

			volumeName := getVolumeName()
			engineName := fmt.Sprintf("%s-e", volumeName)
			engineFrontendName := fmt.Sprintf("%s-ef", volumeName)
			replicaNames := []string{
				fmt.Sprintf("%s-replica-1", volumeName),
				fmt.Sprintf("%s-replica-2", volumeName),
			}
			replicas := make(map[string]*api.Replica)

			defer func() {
				err = spdkCli.EngineFrontendDelete(engineFrontendName)
				c.Assert(err, IsNil)

				err = spdkCli.EngineDelete(engineName)
				c.Assert(err, IsNil)

				for _, replica := range replicas {
					err = spdkCli.ReplicaDelete(replica.Name, true)
					c.Assert(err, IsNil)
				}
			}()

			for _, replicaName := range replicaNames {
				replica, err := spdkCli.ReplicaCreate(replicaName, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
				c.Assert(err, IsNil)
				c.Assert(replica.LvsName, Equals, defaultTestDiskName)
				c.Assert(replica.LvsUUID, Equals, disk.Uuid)
				c.Assert(replica.ErrorMsg, Equals, "")
				c.Assert(replica.State, Equals, types.InstanceStateRunning)
				c.Assert(replica.PortStart, Not(Equals), int32(0))
				c.Assert(replica.Head, NotNil)
				c.Assert(replica.Head.CreationTime, Not(Equals), "")
				c.Assert(replica.Head.Parent, Equals, "")
				replicas[replicaName] = replica
			}

			replicaAddressMap := make(map[string]string)
			replicaModeMap := make(map[string]types.Mode)
			for _, replica := range replicas {
				replicaAddressMap[replica.Name] = net.JoinHostPort(ip, strconv.Itoa(int(replica.PortStart)))
				replicaModeMap[replica.Name] = types.ModeRW
			}

			engine, err := spdkCli.EngineCreate(engineName, volumeName, test.frontend, defaultTestLvolSize, replicaAddressMap, 1, false)
			c.Assert(err, IsNil)
			c.Assert(engine.State, Equals, types.InstanceStateRunning)
			c.Assert(engine.ErrorMsg, Equals, "")
			c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
			c.Assert(engine.ReplicaModeMap, DeepEquals, replicaModeMap)
			assertEngineEndpoint(c, engine.IP, engine.Port, test.expectEngineEndpoint)

			engineList, err := spdkCli.EngineList()
			c.Assert(err, IsNil)
			c.Assert(engineList, HasLen, 1)
			for _, e := range engineList {
				assertEngineEndpoint(c, e.IP, e.Port, test.expectEngineEndpoint)
			}

			replicaList, err := spdkCli.ReplicaList()
			c.Assert(err, IsNil)
			c.Assert(replicaList, HasLen, 2)

			e, err := spdkCli.EngineGet(engineName)
			c.Assert(err, IsNil)
			assertEngineEndpoint(c, e.IP, e.Port, test.expectEngineEndpoint)

			engineFrontend, err := spdkCli.EngineFrontendCreate(engineFrontendName, volumeName, engineName, test.frontend, defaultTestLvolSize,
				net.JoinHostPort(e.IP, strconv.Itoa(int(e.Port))), 0, 0)
			c.Assert(err, IsNil)
			c.Assert(engineFrontend.State, Equals, types.InstanceStateRunning)
			c.Assert(engineFrontend.ErrorMsg, Equals, "")
			c.Assert(engineFrontend.TargetIP, Equals, e.IP)
			c.Assert(engineFrontend.TargetPort, Equals, e.Port)

			test.verifyFrontend(c, volumeName, engineName, e, engineFrontend)
		}()
	}
}

func (s *TestSuite) TestSPDKEngineFrontendSuspendAndResume(c *C) {
	fmt.Println("Testing SPDK engine frontend suspend")

	diskDriverName := "aio"

	ip, err := commonnet.GetAnyExternalIP()
	c.Assert(err, IsNil)
	err = os.Setenv(commonnet.EnvPodIP, ip)
	c.Assert(err, IsNil)

	ctx, cancel := context.WithCancel(context.Background())
	var spdkWg sync.WaitGroup

	defer func() {
		cancel()
		spdkWg.Wait()
	}()

	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	c.Assert(err, IsNil)
	LaunchTestSPDKGRPCServer(ctx, c, ip, ne.Execute, &spdkWg)

	loopDevicePath := PrepareDiskFile(c)
	defer func() {
		CleanupDiskFile(c, loopDevicePath)
	}()

	spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
	c.Assert(err, IsNil)
	defer func() {
		if errClose := spdkCli.Close(); errClose != nil {
			logrus.WithError(errClose).Error("Failed to close SPDK client")
		}
	}()

	disk, err := spdkCli.DiskCreate(defaultTestDiskName, "", loopDevicePath, diskDriverName, int64(defaultTestBlockSize))
	c.Assert(err, IsNil)
	c.Assert(disk, NotNil)

	disk, err = waitForDiskReady(ctx, spdkCli, loopDevicePath, diskDriverName)
	c.Assert(err, IsNil)
	c.Assert(disk.Path, Equals, loopDevicePath)
	c.Assert(disk.Uuid, Not(Equals), "")

	defer func() {
		err := spdkCli.DiskDelete(defaultTestDiskName, disk.Uuid, disk.Path, diskDriverName)
		c.Assert(err, IsNil)

		disk, err = spdkCli.DiskGet(defaultTestDiskName, disk.Path, diskDriverName)
		c.Assert(err, NotNil)
		c.Assert(disk, IsNil)
	}()

	volumeName := getVolumeName()
	engineName := fmt.Sprintf("%s-e", volumeName)
	engineFrontendName := fmt.Sprintf("%s-ef", volumeName)
	replicaNames := []string{
		fmt.Sprintf("%s-replica-1", volumeName),
		fmt.Sprintf("%s-replica-2", volumeName),
	}
	replicas := make(map[string]*api.Replica)

	defer func() {
		err = spdkCli.EngineFrontendDelete(engineFrontendName)
		c.Assert(err, IsNil)

		err = spdkCli.EngineDelete(engineName)
		c.Assert(err, IsNil)

		for _, replica := range replicas {
			err = spdkCli.ReplicaDelete(replica.Name, true)
			c.Assert(err, IsNil)
		}
	}()

	// Create replicas
	for _, replicaName := range replicaNames {
		replica, err := spdkCli.ReplicaCreate(replicaName, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
		c.Assert(err, IsNil)
		c.Assert(replica.LvsName, Equals, defaultTestDiskName)
		c.Assert(replica.LvsUUID, Equals, disk.Uuid)
		c.Assert(replica.ErrorMsg, Equals, "")
		c.Assert(replica.State, Equals, types.InstanceStateRunning)
		c.Assert(replica.PortStart, Not(Equals), int32(0))
		c.Assert(replica.Head, NotNil)
		c.Assert(replica.Head.CreationTime, Not(Equals), "")
		c.Assert(replica.Head.Parent, Equals, "")
		replicas[replicaName] = replica
	}

	// Create replica address map and replica mode map
	replicaAddressMap := make(map[string]string)
	replicaModeMap := make(map[string]types.Mode)
	for _, replica := range replicas {
		replicaAddressMap[replica.Name] = net.JoinHostPort(ip, strconv.Itoa(int(replica.PortStart)))
		replicaModeMap[replica.Name] = types.ModeRW
	}

	// Create engine target
	engine, err := spdkCli.EngineCreate(engineName, volumeName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize, replicaAddressMap, 1, false)
	c.Assert(err, IsNil)
	c.Assert(engine.State, Equals, types.InstanceStateRunning)
	c.Assert(engine.ErrorMsg, Equals, "")
	c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
	c.Assert(engine.ReplicaModeMap, DeepEquals, replicaModeMap)
	c.Assert(engine.IP, Not(Equals), "")
	c.Assert(engine.Port, Not(Equals), int32(0))

	engineList, err := spdkCli.EngineList()
	c.Assert(err, IsNil)
	c.Assert(engineList, HasLen, 1)
	for _, e := range engineList {
		c.Assert(e.IP, Not(Equals), "")
		c.Assert(e.Port, Not(Equals), int32(0))
	}

	replicaList, err := spdkCli.ReplicaList()
	c.Assert(err, IsNil)
	c.Assert(replicaList, HasLen, 2)

	e, err := spdkCli.EngineGet(engineName)
	c.Assert(err, IsNil)
	c.Assert(e.IP, Not(Equals), "")
	c.Assert(e.Port, Not(Equals), int32(0))

	// Create engine frontend (NVMe/TCP initiator)
	engineFrontend, err := spdkCli.EngineFrontendCreate(engineFrontendName, volumeName, engineName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize,
		net.JoinHostPort(e.IP, strconv.Itoa(int(e.Port))), 0, 0)

	c.Assert(err, IsNil)
	c.Assert(engineFrontend.State, Equals, types.InstanceStateRunning)
	c.Assert(engineFrontend.ErrorMsg, Equals, "")
	c.Assert(engineFrontend.TargetIP, Equals, e.IP)
	c.Assert(engineFrontend.TargetPort, Equals, e.Port)

	endpoint := helperutil.GetLonghornDevicePath(volumeName)
	c.Assert(engineFrontend.Endpoint, Equals, endpoint)

	// Format block device to check if the engine frontend is working
	err = formatBlockDevice(endpoint, "ext4")
	c.Assert(err, IsNil)

	// Write a known 1MB zero pattern at offset 400MB (beyond the background dd's range)
	// and compute its checksum for later verification.
	_, err = ne.Execute(nil, "dd",
		[]string{
			"if=/dev/zero",
			fmt.Sprintf("of=%s", endpoint),
			"bs=1M", "count=1", "seek=400", "oflag=direct", "conv=notrunc", "status=none",
		},
		10*time.Second,
	)
	c.Assert(err, IsNil)

	checksumBefore, err := ne.Execute(nil, "sh",
		[]string{
			"-c",
			fmt.Sprintf("dd if=%s bs=1M count=1 skip=400 iflag=direct status=none | md5sum", endpoint),
		},
		10*time.Second,
	)
	c.Assert(err, IsNil)
	c.Assert(strings.TrimSpace(checksumBefore), Not(Equals), "")

	// Start a slow background write using direct IO + dsync.
	// This ensures IO is still in progress when we call Suspend.
	// We write ~10MB (2500 × 4K) which is slow enough with dsync to still be in-flight
	// after 1 second, but fast enough to complete shortly after resume.
	ddDone := make(chan error, 1)
	go func() {
		_, err := ne.Execute(nil, "dd",
			[]string{
				"if=/dev/urandom",
				fmt.Sprintf("of=%s", endpoint),
				"bs=4k", "count=2500", "oflag=direct,dsync", "conv=notrunc", "status=none",
			},
			30*time.Second,
		)
		ddDone <- err
	}()

	// Give dd time to start writing before suspending
	time.Sleep(1 * time.Second)

	// Suspend engine frontend — this freezes the dm-device, queuing all in-flight and new IO
	err = spdkCli.EngineFrontendSuspend(engineFrontend.Name)
	c.Assert(err, IsNil)

	// Verify that new IO is blocked while the engine frontend is suspended.
	// The dd command should time out because the dm-device suspend queues all new IO.
	// We use oflag=direct to bypass the page cache, ensuring the IO hits the dm-device
	// and gets queued by suspend (without direct IO, dd returns immediately after writing
	// to the page cache).
	_, err = ne.Execute(nil, "dd",
		[]string{
			"if=/dev/urandom",
			fmt.Sprintf("of=%s", endpoint),
			"bs=1M", "count=1", "seek=0", "oflag=direct", "conv=notrunc", "status=none",
		},
		10*time.Second,
	)
	c.Assert(err, NotNil)

	// Resume engine frontend — this unblocks the queued IO
	err = spdkCli.EngineFrontendResume(engineFrontend.Name)
	c.Assert(err, IsNil)

	// The background dd should now complete (successfully or with a write error,
	// but it should no longer be stuck). We wait up to 60 seconds to account for
	// the dd's Execute timeout plus residual IO after resume.
	select {
	case <-ddDone:
	case <-time.After(60 * time.Second):
		c.Fatal("background dd did not complete after resume")
	}

	// Verify data integrity: the checksum of the region at offset 400MB should be unchanged
	// after the suspend/resume cycle.
	checksumAfter, err := ne.Execute(nil, "sh",
		[]string{
			"-c",
			fmt.Sprintf("dd if=%s bs=1M count=1 skip=400 iflag=direct status=none | md5sum", endpoint),
		},
		10*time.Second,
	)
	c.Assert(err, IsNil)
	c.Assert(strings.TrimSpace(checksumAfter), Equals, strings.TrimSpace(checksumBefore))
}

func (s *TestSuite) TestSPDKEngineFrontendExpand(c *C) {
	fmt.Println("Testing SPDK engine frontend expand")

	diskDriverName := "aio"

	ip, err := commonnet.GetAnyExternalIP()
	c.Assert(err, IsNil)
	err = os.Setenv(commonnet.EnvPodIP, ip)
	c.Assert(err, IsNil)

	ctx, cancel := context.WithCancel(context.Background())
	var spdkWg sync.WaitGroup

	defer func() {
		cancel()
		spdkWg.Wait()
	}()

	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	c.Assert(err, IsNil)
	LaunchTestSPDKGRPCServer(ctx, c, ip, ne.Execute, &spdkWg)

	loopDevicePath := PrepareDiskFile(c)
	defer func() {
		CleanupDiskFile(c, loopDevicePath)
	}()

	spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
	c.Assert(err, IsNil)
	defer func() {
		if errClose := spdkCli.Close(); errClose != nil {
			logrus.WithError(errClose).Error("Failed to close SPDK client")
		}
	}()

	disk, err := spdkCli.DiskCreate(defaultTestDiskName, "", loopDevicePath, diskDriverName, int64(defaultTestBlockSize))
	c.Assert(err, IsNil)
	c.Assert(disk, NotNil)

	disk, err = waitForDiskReady(ctx, spdkCli, loopDevicePath, diskDriverName)
	c.Assert(err, IsNil)
	c.Assert(disk.Path, Equals, loopDevicePath)
	c.Assert(disk.Uuid, Not(Equals), "")

	defer func() {
		err := spdkCli.DiskDelete(defaultTestDiskName, disk.Uuid, disk.Path, diskDriverName)
		c.Assert(err, IsNil)

		disk, err = spdkCli.DiskGet(defaultTestDiskName, disk.Path, diskDriverName)
		c.Assert(err, NotNil)
		c.Assert(disk, IsNil)
	}()

	volumeName := getVolumeName()
	engineName := fmt.Sprintf("%s-e", volumeName)
	engineFrontendName := fmt.Sprintf("%s-ef", volumeName)
	replicaNames := []string{
		fmt.Sprintf("%s-replica-1", volumeName),
		fmt.Sprintf("%s-replica-2", volumeName),
	}
	replicas := make(map[string]*api.Replica)

	defer func() {
		err = spdkCli.EngineFrontendDelete(engineFrontendName)
		c.Assert(err, IsNil)

		err = spdkCli.EngineDelete(engineName)
		c.Assert(err, IsNil)

		for _, replica := range replicas {
			err = spdkCli.ReplicaDelete(replica.Name, true)
			c.Assert(err, IsNil)
		}
	}()

	// Create replicas
	for _, replicaName := range replicaNames {
		replica, err := spdkCli.ReplicaCreate(replicaName, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
		c.Assert(err, IsNil)
		c.Assert(replica.LvsName, Equals, defaultTestDiskName)
		c.Assert(replica.LvsUUID, Equals, disk.Uuid)
		c.Assert(replica.ErrorMsg, Equals, "")
		c.Assert(replica.State, Equals, types.InstanceStateRunning)
		c.Assert(replica.PortStart, Not(Equals), int32(0))
		c.Assert(replica.Head, NotNil)
		c.Assert(replica.Head.CreationTime, Not(Equals), "")
		c.Assert(replica.Head.Parent, Equals, "")
		replicas[replicaName] = replica
	}

	// Create replica address map and replica mode map
	replicaAddressMap := make(map[string]string)
	replicaModeMap := make(map[string]types.Mode)
	for _, replica := range replicas {
		replicaAddressMap[replica.Name] = net.JoinHostPort(ip, strconv.Itoa(int(replica.PortStart)))
		replicaModeMap[replica.Name] = types.ModeRW
	}

	// Create engine target
	engine, err := spdkCli.EngineCreate(engineName, volumeName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize, replicaAddressMap, 1, false)
	c.Assert(err, IsNil)
	c.Assert(engine.State, Equals, types.InstanceStateRunning)
	c.Assert(engine.ErrorMsg, Equals, "")
	c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
	c.Assert(engine.ReplicaModeMap, DeepEquals, replicaModeMap)
	c.Assert(engine.IP, Not(Equals), "")
	c.Assert(engine.Port, Not(Equals), int32(0))

	engineList, err := spdkCli.EngineList()
	c.Assert(err, IsNil)
	c.Assert(engineList, HasLen, 1)
	for _, e := range engineList {
		c.Assert(e.IP, Not(Equals), "")
		c.Assert(e.Port, Not(Equals), int32(0))
	}

	replicaList, err := spdkCli.ReplicaList()
	c.Assert(err, IsNil)
	c.Assert(replicaList, HasLen, 2)

	e, err := spdkCli.EngineGet(engineName)
	c.Assert(err, IsNil)
	c.Assert(e.IP, Not(Equals), "")
	c.Assert(e.Port, Not(Equals), int32(0))

	// Create engine frontend (NVMe/TCP initiator)
	engineFrontend, err := spdkCli.EngineFrontendCreate(engineFrontendName, volumeName, engineName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize,
		net.JoinHostPort(e.IP, strconv.Itoa(int(e.Port))), 0, 0)

	c.Assert(err, IsNil)
	c.Assert(engineFrontend.State, Equals, types.InstanceStateRunning)
	c.Assert(engineFrontend.ErrorMsg, Equals, "")
	c.Assert(engineFrontend.TargetIP, Equals, e.IP)
	c.Assert(engineFrontend.TargetPort, Equals, e.Port)

	endpoint := helperutil.GetLonghornDevicePath(volumeName)
	c.Assert(engineFrontend.Endpoint, Equals, endpoint)

	size := GetBlockDeviceSize(ne, endpoint)
	c.Assert(size, Equals, int64(defaultTestLvolSize))

	// Format block device to check if the engine frontend is working
	err = formatBlockDevice(endpoint, "ext4")
	c.Assert(err, IsNil)

	// Suspend and resume engine frontend
	newTestLvolSize := defaultTestLvolSize * 2
	err = spdkCli.EngineFrontendExpand(ctx, engineFrontend.Name, newTestLvolSize)
	c.Assert(err, IsNil)

	// Format block device to check if the engine frontend is working
	err = formatBlockDevice(endpoint, "ext4")
	c.Assert(err, IsNil)

	size = GetBlockDeviceSize(ne, endpoint)
	c.Assert(size, Equals, int64(newTestLvolSize))
}

func (s *TestSuite) TestSPDKEngineSnapshotCreateAndDelete(c *C) {
	fmt.Println("Testing SPDK engine snapshot create and delete")

	diskDriverName := "aio"

	ip, err := commonnet.GetAnyExternalIP()
	c.Assert(err, IsNil)
	err = os.Setenv(commonnet.EnvPodIP, ip)
	c.Assert(err, IsNil)

	ctx, cancel := context.WithCancel(context.Background())
	var spdkWg sync.WaitGroup
	defer func() {
		cancel()
		spdkWg.Wait()
	}()

	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	c.Assert(err, IsNil)
	LaunchTestSPDKGRPCServer(ctx, c, ip, ne.Execute, &spdkWg)

	loopDevicePath := PrepareDiskFile(c)
	defer func() {
		CleanupDiskFile(c, loopDevicePath)
	}()

	spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
	c.Assert(err, IsNil)
	defer func() {
		if errClose := spdkCli.Close(); errClose != nil {
			logrus.WithError(errClose).Error("Failed to close SPDK client")
		}
	}()

	disk, err := spdkCli.DiskCreate(defaultTestDiskName, "", loopDevicePath, diskDriverName, int64(defaultTestBlockSize))
	c.Assert(err, IsNil)
	c.Assert(disk, NotNil)

	disk, err = waitForDiskReady(ctx, spdkCli, loopDevicePath, diskDriverName)
	c.Assert(err, IsNil)
	c.Assert(disk.Path, Equals, loopDevicePath)
	c.Assert(disk.Uuid, Not(Equals), "")

	defer func() {
		err := spdkCli.DiskDelete(defaultTestDiskName, disk.Uuid, disk.Path, diskDriverName)
		c.Assert(err, IsNil)
	}()

	volumeName := getVolumeName()
	engineName := fmt.Sprintf("%s-e", volumeName)
	engineFrontendName := fmt.Sprintf("%s-ef", volumeName)
	replicaNames := []string{
		fmt.Sprintf("%s-replica-1", volumeName),
		fmt.Sprintf("%s-replica-2", volumeName),
	}
	replicas := make(map[string]*api.Replica)

	defer func() {
		err = spdkCli.EngineFrontendDelete(engineFrontendName)
		c.Assert(err, IsNil)

		err = spdkCli.EngineDelete(engineName)
		c.Assert(err, IsNil)

		for _, replica := range replicas {
			err = spdkCli.ReplicaDelete(replica.Name, true)
			c.Assert(err, IsNil)
		}
	}()

	for _, replicaName := range replicaNames {
		replica, err := spdkCli.ReplicaCreate(replicaName, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
		c.Assert(err, IsNil)
		replicas[replicaName] = replica
	}

	replicaAddressMap := make(map[string]string)
	replicaModeMap := make(map[string]types.Mode)
	for _, replica := range replicas {
		replicaAddressMap[replica.Name] = net.JoinHostPort(ip, strconv.Itoa(int(replica.PortStart)))
		replicaModeMap[replica.Name] = types.ModeRW
	}

	engine, err := spdkCli.EngineCreate(engineName, volumeName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize, replicaAddressMap, 1, false)
	c.Assert(err, IsNil)
	c.Assert(engine.State, Equals, types.InstanceStateRunning)
	c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
	c.Assert(engine.ReplicaModeMap, DeepEquals, replicaModeMap)

	engineFrontend, err := spdkCli.EngineFrontendCreate(engineFrontendName, volumeName, engineName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize,
		net.JoinHostPort(ip, strconv.Itoa(int(engine.Port))), 0, 0)
	c.Assert(err, IsNil)
	c.Assert(engineFrontend, NotNil)

	snapshotName := "snapshot-1"
	retSnapshotName, err := spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName)
	c.Assert(err, IsNil)
	c.Assert(retSnapshotName, Equals, snapshotName)

	engineAfterCreate, err := spdkCli.EngineGet(engineName)
	c.Assert(err, IsNil)
	_, exists := engineAfterCreate.Snapshots[snapshotName]
	c.Assert(exists, Equals, true)

	err = spdkCli.EngineFrontendSnapshotDelete(engineFrontendName, snapshotName)
	c.Assert(err, IsNil)

	engineAfterDelete, err := spdkCli.EngineGet(engineName)
	c.Assert(err, IsNil)
	_, exists = engineAfterDelete.Snapshots[snapshotName]
	c.Assert(exists, Equals, false)

	err = spdkCli.EngineSnapshotDelete(engineName, snapshotName)
	c.Assert(err, NotNil)
}

// TestSPDKEngineFrontendSnapshotRevert validates both negative and positive revert paths.
// Revert should fail with non-empty frontend and succeed with FrontendEmpty.
func (s *TestSuite) TestSPDKEngineFrontendSnapshotRevert(c *C) {
	fmt.Println("Testing SPDK engine frontend snapshot revert")

	tests := []struct {
		name               string
		engineFrontendType string
		expectErr          bool
		errContains        string
	}{
		{
			name:               "revert rejected for spdk-tcp-blockdev",
			engineFrontendType: types.FrontendSPDKTCPBlockdev,
			expectErr:          true,
			errContains:        "invalid frontend",
		},
		// {
		// 	name:               "revert rejected for spdk-tcp-nvmf",
		// 	engineFrontendType: types.FrontendSPDKTCPNvmf,
		// 	expectErr:          true,
		// 	errContains:        "invalid frontend",
		// },
		// {
		// 	name:               "revert rejected for ublk",
		// 	engineFrontendType: types.FrontendUBLK,
		// 	expectErr:          true,
		// 	errContains:        "invalid frontend",
		// },
		// {
		// 	name:               "revert succeeds for frontend-empty",
		// 	engineFrontendType: types.FrontendEmpty,
		// 	expectErr:          false,
		// },
	}

	for _, tc := range tests {
		c.Logf("testing %s", tc.name)

		diskDriverName := "aio"

		ip, err := commonnet.GetAnyExternalIP()
		c.Assert(err, IsNil)
		err = os.Setenv(commonnet.EnvPodIP, ip)
		c.Assert(err, IsNil)

		ctx, cancel := context.WithCancel(context.Background())
		var spdkWg sync.WaitGroup
		func() {
			defer func() {
				cancel()
				spdkWg.Wait()
			}()

			ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
			c.Assert(err, IsNil)
			LaunchTestSPDKGRPCServer(ctx, c, ip, ne.Execute, &spdkWg)

			loopDevicePath := PrepareDiskFile(c)
			defer func() {
				CleanupDiskFile(c, loopDevicePath)
			}()

			spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
			c.Assert(err, IsNil)
			defer func() {
				if errClose := spdkCli.Close(); errClose != nil {
					logrus.WithError(errClose).Error("Failed to close SPDK client")
				}
			}()

			disk, err := spdkCli.DiskCreate(defaultTestDiskName, "", loopDevicePath, diskDriverName, int64(defaultTestBlockSize))
			c.Assert(err, IsNil)
			c.Assert(disk, NotNil)

			disk, err = waitForDiskReady(ctx, spdkCli, loopDevicePath, diskDriverName)
			c.Assert(err, IsNil)
			c.Assert(disk.Path, Equals, loopDevicePath)
			c.Assert(disk.Uuid, Not(Equals), "")

			defer func() {
				err := spdkCli.DiskDelete(defaultTestDiskName, disk.Uuid, disk.Path, diskDriverName)
				c.Assert(err, IsNil)
			}()

			volumeName := fmt.Sprintf("test-frontend-revert-vol-%s", time.Now().Format("20060102150405"))
			engineName := fmt.Sprintf("%s-e", volumeName)
			engineFrontendName := fmt.Sprintf("%s-ef", volumeName)
			replicaNames := []string{
				fmt.Sprintf("%s-replica-1", volumeName),
				fmt.Sprintf("%s-replica-2", volumeName),
			}
			replicas := make(map[string]*api.Replica)

			defer func() {
				err = spdkCli.EngineFrontendDelete(engineFrontendName)
				c.Assert(err, IsNil)

				err = spdkCli.EngineDelete(engineName)
				c.Assert(err, IsNil)

				for _, replica := range replicas {
					err = spdkCli.ReplicaDelete(replica.Name, true)
					c.Assert(err, IsNil)
				}
			}()

			for _, replicaName := range replicaNames {
				replica, err := spdkCli.ReplicaCreate(replicaName, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
				c.Assert(err, IsNil)
				replicas[replicaName] = replica
			}

			replicaAddressMap := make(map[string]string)
			for _, replica := range replicas {
				replicaAddressMap[replica.Name] = net.JoinHostPort(ip, strconv.Itoa(int(replica.PortStart)))
			}

			engine, err := spdkCli.EngineCreate(engineName, volumeName, tc.engineFrontendType, defaultTestLvolSize, replicaAddressMap, 1, false)
			c.Assert(err, IsNil)
			c.Assert(engine, NotNil)
			if engine.State != types.InstanceStateRunning {
				c.Skip(fmt.Sprintf("skip %s: engine is not running (state=%s, error=%s)", tc.name, engine.State, engine.ErrorMsg))
			}

			engineFrontendTargetAddress := net.JoinHostPort(ip, strconv.Itoa(int(engine.Port)))

			engineFrontend, err := spdkCli.EngineFrontendCreate(engineFrontendName, volumeName, engineName, tc.engineFrontendType, defaultTestLvolSize,
				engineFrontendTargetAddress, 0, 0)
			c.Assert(err, IsNil)
			c.Assert(engineFrontend, NotNil)

			snapshotName1 := "snapshot-1"
			snapshotName2 := "snapshot-2"

			retSnapshotName, err := spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName1)
			c.Assert(err, IsNil)
			c.Assert(retSnapshotName, Equals, snapshotName1)

			retSnapshotName, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName2)
			c.Assert(err, IsNil)
			c.Assert(retSnapshotName, Equals, snapshotName2)

			err = spdkCli.EngineFrontendSnapshotRevert(engineFrontendName, snapshotName1)
			if tc.expectErr {
				c.Assert(err, NotNil)
				c.Assert(strings.Contains(err.Error(), tc.errContains), Equals, true)
			} else {
				c.Assert(err, IsNil)
			}

			engineAfterRevert, err := spdkCli.EngineGet(engineName)
			c.Assert(err, IsNil)
			c.Assert(engineAfterRevert.State, Equals, types.InstanceStateRunning)
			_, exists := engineAfterRevert.Snapshots[snapshotName1]
			c.Assert(exists, Equals, true)
		}()
	}
}

type runtimeMonitoringTestEnv struct {
	ctx        context.Context
	spdkCli    *client.SPDKClient
	rawSPDKCli *helperclient.Client
	disk       *spdkrpc.Disk
}

func isReplicaNotFound(err error) bool {
	if err == nil {
		return false
	}
	if grpcstatus.Code(err) == grpccodes.NotFound {
		return true
	}
	return strings.Contains(err.Error(), "cannot find replica")
}

func monitoringRetryOpts(ctx context.Context, attempts uint) []retry.Option {
	return []retry.Option{
		retry.Context(ctx),
		retry.Attempts(attempts),
		retry.DelayType(retry.FixedDelay),
		retry.LastErrorOnly(true),
		retry.Delay(server.MonitorInterval),
	}
}

func waitForSPDKTargetStopped() error {
	return waitForSPDKTargetDaemonStoppedWithTimeout(6*time.Second, spdkTargetStopPollInterval)
}

func withRuntimeMonitoringTestEnv(c *C, diskDriverName string, testFn func(env *runtimeMonitoringTestEnv)) {
	runtimeMonitoringTestMu.Lock()
	defer runtimeMonitoringTestMu.Unlock()

	// Ensure no lingering target process from a previous test case.
	err := waitForSPDKTargetStopped()
	c.Assert(err, IsNil)

	ip, err := commonnet.GetAnyExternalIP()
	c.Assert(err, IsNil)
	err = os.Setenv(commonnet.EnvPodIP, ip)
	c.Assert(err, IsNil)

	ctx, cancel := context.WithCancel(context.Background())
	var spdkWg sync.WaitGroup

	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	c.Assert(err, IsNil)
	LaunchTestSPDKGRPCServer(ctx, c, ip, ne.Execute, &spdkWg)

	loopDevicePath := PrepareDiskFile(c)
	spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
	c.Assert(err, IsNil)
	rawSPDKCli, err := helperclient.NewClient(context.Background())
	c.Assert(err, IsNil)

	disk, err := spdkCli.DiskCreate(defaultTestDiskName, "", loopDevicePath, diskDriverName, int64(defaultTestBlockSize))
	c.Assert(err, IsNil)
	c.Assert(disk, NotNil)
	disk, err = waitForDiskReady(ctx, spdkCli, loopDevicePath, diskDriverName)
	c.Assert(err, IsNil)

	defer func() {
		err := rawSPDKCli.Close()
		c.Assert(err, IsNil)
		err = spdkCli.DiskDelete(defaultTestDiskName, disk.Uuid, disk.Path, diskDriverName)
		c.Assert(err, IsNil)
		err = spdkCli.Close()
		c.Assert(err, IsNil)
		CleanupDiskFile(c, loopDevicePath)
		cancel()
		spdkWg.Wait()
		err = waitForSPDKTargetStopped()
		c.Assert(err, IsNil)
	}()

	testFn(&runtimeMonitoringTestEnv{
		ctx:        ctx,
		spdkCli:    spdkCli,
		rawSPDKCli: rawSPDKCli,
		disk:       disk,
	})
}

func (s *TestSuite) TestRuntimeMonitoringVerifyMultipleReplicasReconstruction(c *C) {
	fmt.Println("Testing runtime monitoring verify() for multiple replicas reconstruction")
	withRuntimeMonitoringTestEnv(c, "aio", func(env *runtimeMonitoringTestEnv) {
		replicaNames := []string{
			fmt.Sprintf("matrix-r-%s", strings.ReplaceAll(util.UUID(), "-", "")[:8]),
			fmt.Sprintf("matrix-r-%s", strings.ReplaceAll(util.UUID(), "-", "")[:8]),
			fmt.Sprintf("matrix-r-%s", strings.ReplaceAll(util.UUID(), "-", "")[:8]),
		}

		for _, name := range replicaNames {
			_, err := env.rawSPDKCli.BdevLvolCreate("", env.disk.Uuid, name, util.BytesToMiB(defaultTestLvolSize), "", true)
			c.Assert(err, IsNil)
			defer func(replicaName string) {
				_ = env.spdkCli.ReplicaDelete(replicaName, true)
				_, _ = env.rawSPDKCli.BdevLvolDelete(fmt.Sprintf("%s/%s", env.disk.Name, replicaName))
			}(name)
		}

		err := retry.Do(func() error {
			for _, name := range replicaNames {
				replica, err := env.spdkCli.ReplicaGet(name)
				if err != nil {
					return err
				}
				if replica.State != types.InstanceStateStopped {
					return fmt.Errorf("replica %s state is %s, expected %s", replica.Name, replica.State, types.InstanceStateStopped)
				}
			}
			return nil
		}, monitoringRetryOpts(env.ctx, 8)...)
		c.Assert(err, IsNil)

		replicaMap, err := env.spdkCli.ReplicaList()
		c.Assert(err, IsNil)
		for _, name := range replicaNames {
			_, exists := replicaMap[name]
			c.Assert(exists, Equals, true)
		}
	})
}

func (s *TestSuite) TestRuntimeMonitoringVerifyMixedValidAndInvalidLvolNames(c *C) {
	fmt.Println("Testing runtime monitoring verify() with mixed valid and invalid lvol names")
	withRuntimeMonitoringTestEnv(c, "aio", func(env *runtimeMonitoringTestEnv) {
		validReplicaName := fmt.Sprintf("matrix-r-%s", strings.ReplaceAll(util.UUID(), "-", "")[:8])
		invalidNames := []string{
			"matrix-invalid",
			"matrix-r-123",
			"matrix-r-123456789",
		}

		// Mismatching-size lvol: valid replica name but half the expected size.
		// This passes the IsProbablyReplicaName filter and gets reconstructed by
		// monitoring, but with a specSize that doesn't match defaultTestLvolSize.
		mismatchSizeName := fmt.Sprintf("matrix-r-%s", strings.ReplaceAll(util.UUID(), "-", "")[:8])
		mismatchSizeInMiB := util.BytesToMiB(defaultTestLvolSize) / 2

		allNames := append([]string{validReplicaName}, invalidNames...)

		// Create valid + invalid-name lvols with defaultTestLvolSize
		var createWg sync.WaitGroup
		createErrs := make([]error, len(allNames))
		for i, name := range allNames {
			createWg.Add(1)
			go func(idx int, lvolName string) {
				defer createWg.Done()
				_, createErrs[idx] = env.rawSPDKCli.BdevLvolCreate("", env.disk.Uuid, lvolName, util.BytesToMiB(defaultTestLvolSize), "", true)
			}(i, name)
		}
		createWg.Wait()
		for i, name := range allNames {
			c.Assert(createErrs[i], IsNil, Commentf("failed to create lvol %s", name))
			defer func(replicaName string) {
				_ = env.spdkCli.ReplicaDelete(replicaName, true)
				_, _ = env.rawSPDKCli.BdevLvolDelete(fmt.Sprintf("%s/%s", env.disk.Name, replicaName))
			}(name)
		}

		// Create mismatching-size lvol separately with half the expected size
		_, err := env.rawSPDKCli.BdevLvolCreate("", env.disk.Uuid, mismatchSizeName, mismatchSizeInMiB, "", true)
		c.Assert(err, IsNil)
		defer func() {
			_ = env.spdkCli.ReplicaDelete(mismatchSizeName, true)
			_, _ = env.rawSPDKCli.BdevLvolDelete(fmt.Sprintf("%s/%s", env.disk.Name, mismatchSizeName))
		}()

		// Wait for both the valid replica and mismatching-size replica to be detected
		err = retry.Do(func() error {
			replica, err := env.spdkCli.ReplicaGet(validReplicaName)
			if err != nil {
				return err
			}
			if replica.State != types.InstanceStateStopped {
				return fmt.Errorf("replica %s state is %s, expected %s", replica.Name, replica.State, types.InstanceStateStopped)
			}
			mismatchReplica, err := env.spdkCli.ReplicaGet(mismatchSizeName)
			if err != nil {
				return err
			}
			if mismatchReplica.State != types.InstanceStateStopped {
				return fmt.Errorf("replica %s state is %s, expected %s", mismatchReplica.Name, mismatchReplica.State, types.InstanceStateStopped)
			}
			return nil
		}, monitoringRetryOpts(env.ctx, 8)...)
		c.Assert(err, IsNil)

		// Verify the mismatching-size replica is reconstructed with the wrong specSize.
		// Monitoring derives specSize from the lvol itself, so it doesn't detect the mismatch.
		mismatchReplica, err := env.spdkCli.ReplicaGet(mismatchSizeName)
		c.Assert(err, IsNil)
		c.Assert(mismatchReplica.SpecSize, Equals, mismatchSizeInMiB*1024*1024)
		c.Assert(mismatchReplica.SpecSize, Not(Equals), defaultTestLvolSize)

		// Verify invalid-name lvols are not found concurrently
		var verifyWg sync.WaitGroup
		verifyResults := make([]bool, len(invalidNames))
		for i, invalidName := range invalidNames {
			verifyWg.Add(1)
			go func(idx int, name string) {
				defer verifyWg.Done()
				_, err := env.spdkCli.ReplicaGet(name)
				verifyResults[idx] = isReplicaNotFound(err)
			}(i, invalidName)
		}
		verifyWg.Wait()
		for i, invalidName := range invalidNames {
			c.Assert(verifyResults[i], Equals, true, Commentf("expected replica %s to be not found", invalidName))
		}
	})
}

func (s *TestSuite) TestRuntimeMonitoringVerifyStaleCacheRemoval(c *C) {
	fmt.Println("Testing runtime monitoring verify() stale cache removal")
	withRuntimeMonitoringTestEnv(c, "aio", func(env *runtimeMonitoringTestEnv) {
		replicaName := fmt.Sprintf("matrix-r-%s", strings.ReplaceAll(util.UUID(), "-", "")[:8])
		replicaAlias := fmt.Sprintf("%s/%s", env.disk.Name, replicaName)

		_, err := env.rawSPDKCli.BdevLvolCreate("", env.disk.Uuid, replicaName, util.BytesToMiB(defaultTestLvolSize), "", true)
		c.Assert(err, IsNil)
		defer func() {
			_ = env.spdkCli.ReplicaDelete(replicaName, true)
			_, _ = env.rawSPDKCli.BdevLvolDelete(replicaAlias)
		}()

		err = retry.Do(func() error {
			_, err := env.spdkCli.ReplicaGet(replicaName)
			return err
		}, monitoringRetryOpts(env.ctx, 8)...)
		c.Assert(err, IsNil)

		_, err = env.rawSPDKCli.BdevLvolDelete(replicaAlias)
		c.Assert(err, IsNil)

		err = retry.Do(func() error {
			_, err := env.spdkCli.ReplicaGet(replicaName)
			if isReplicaNotFound(err) {
				return nil
			}
			if err == nil {
				return fmt.Errorf("replica %s still exists in cache", replicaName)
			}
			return err
		}, monitoringRetryOpts(env.ctx, 8)...)
		c.Assert(err, IsNil)
	})
}

func (s *TestSuite) TestRuntimeMonitoringVerifySkipRebuildingAndCloningLvol(c *C) {
	fmt.Println("Testing runtime monitoring verify() skips rebuilding and cloning lvols")
	withRuntimeMonitoringTestEnv(c, "aio", func(env *runtimeMonitoringTestEnv) {
		baseReplicaName := fmt.Sprintf("matrix-r-%s", strings.ReplaceAll(util.UUID(), "-", "")[:8])
		rebuildingLvolName := server.GetReplicaRebuildingLvolName(baseReplicaName)
		cloningLvolName := server.GetReplicaCloningLvolName(baseReplicaName)

		for _, name := range []string{rebuildingLvolName, cloningLvolName} {
			_, err := env.rawSPDKCli.BdevLvolCreate("", env.disk.Uuid, name, util.BytesToMiB(defaultTestLvolSize), "", true)
			c.Assert(err, IsNil)
			defer func(replicaName string) {
				_ = env.spdkCli.ReplicaDelete(replicaName, true)
				_, _ = env.rawSPDKCli.BdevLvolDelete(fmt.Sprintf("%s/%s", env.disk.Name, replicaName))
			}(name)
		}

		err := retry.Do(func() error {
			for _, name := range []string{rebuildingLvolName, cloningLvolName, baseReplicaName} {
				_, err := env.spdkCli.ReplicaGet(name)
				if !isReplicaNotFound(err) {
					if err == nil {
						return fmt.Errorf("lvol %s was unexpectedly reconstructed as replica", name)
					}
					return err
				}
			}
			return nil
		}, monitoringRetryOpts(env.ctx, 4)...)
		c.Assert(err, IsNil)
	})
}

func (s *TestSuite) TestRuntimeMonitoringVerifyCleanupOrphanBackingImageTempHead(c *C) {
	fmt.Println("Testing runtime monitoring verify() cleans up orphan backing image temp head")
	withRuntimeMonitoringTestEnv(c, "aio", func(env *runtimeMonitoringTestEnv) {
		backingImageName := fmt.Sprintf("matrix-bi-%s", strings.ReplaceAll(util.UUID(), "-", "")[:8])
		tempHeadName := server.GetBackingImageTempHeadLvolName(backingImageName, env.disk.Uuid)
		tempHeadAlias := fmt.Sprintf("%s/%s", env.disk.Name, tempHeadName)

		_, err := env.rawSPDKCli.BdevLvolCreate("", env.disk.Uuid, tempHeadName, util.BytesToMiB(defaultTestLvolSize), "", true)
		c.Assert(err, IsNil)
		defer func() {
			_, _ = env.rawSPDKCli.BdevLvolDelete(tempHeadAlias)
		}()

		err = retry.Do(func() error {
			bdevs, err := env.rawSPDKCli.BdevGetBdevs("", 0)
			if err != nil {
				return err
			}
			for _, bdev := range bdevs {
				for _, alias := range bdev.Aliases {
					if alias == tempHeadAlias {
						return fmt.Errorf("orphan backing image temp head %s still exists", tempHeadAlias)
					}
				}
			}
			return nil
		}, monitoringRetryOpts(env.ctx, 8)...)
		c.Assert(err, IsNil)
	})
}

// TestSPDKEngineCreateWithSalvageRequested validates salvage flow during engine creation.
// The test creates two replicas, writes random data only to the first replica so its head
// actual size becomes larger, then calls EngineCreate with salvageRequested=true.
// It verifies the engine keeps only the replica with the largest head actual size as the
// salvage candidate.
func (s *TestSuite) TestSPDKEngineCreateWithSalvageRequested(c *C) {
	fmt.Println("Testing SPDK engine create with salvageRequested=true")

	diskDriverName := "aio"

	ip, err := commonnet.GetAnyExternalIP()
	c.Assert(err, IsNil)
	err = os.Setenv(commonnet.EnvPodIP, ip)
	c.Assert(err, IsNil)

	ctx, cancel := context.WithCancel(context.Background())
	var spdkWg sync.WaitGroup
	defer func() {
		cancel()
		spdkWg.Wait()
	}()

	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	c.Assert(err, IsNil)
	LaunchTestSPDKGRPCServer(ctx, c, ip, ne.Execute, &spdkWg)

	loopDevicePath := PrepareDiskFile(c)
	defer func() {
		CleanupDiskFile(c, loopDevicePath)
	}()

	spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
	c.Assert(err, IsNil)
	defer func() {
		if errClose := spdkCli.Close(); errClose != nil {
			logrus.WithError(errClose).Error("Failed to close SPDK client")
		}
	}()

	disk, err := spdkCli.DiskCreate(defaultTestDiskName, "", loopDevicePath, diskDriverName, int64(defaultTestBlockSize))
	c.Assert(err, IsNil)
	c.Assert(disk, NotNil)

	disk, err = waitForDiskReady(ctx, spdkCli, loopDevicePath, diskDriverName)
	c.Assert(err, IsNil)
	c.Assert(disk.Path, Equals, loopDevicePath)
	c.Assert(disk.Uuid, Not(Equals), "")

	defer func() {
		err := spdkCli.DiskDelete(defaultTestDiskName, disk.Uuid, disk.Path, diskDriverName)
		c.Assert(err, IsNil)
	}()

	volumeName := fmt.Sprintf("test-salvage-vol-%s", time.Now().Format("20060102150405"))
	engineName := fmt.Sprintf("%s-e", volumeName)
	engineFrontendName := fmt.Sprintf("%s-ef", volumeName)
	replicaNames := []string{
		fmt.Sprintf("%s-replica-1", volumeName),
		fmt.Sprintf("%s-replica-2", volumeName),
	}
	replicas := make(map[string]*api.Replica)

	defer func() {
		err = spdkCli.EngineFrontendDelete(engineFrontendName)
		c.Assert(err, IsNil)

		err = spdkCli.EngineDelete(engineName)
		c.Assert(err, IsNil)

		for _, replica := range replicas {
			err = spdkCli.ReplicaDelete(replica.Name, true)
			c.Assert(err, IsNil)
		}
	}()

	for _, replicaName := range replicaNames {
		replica, err := spdkCli.ReplicaCreate(replicaName, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
		c.Assert(err, IsNil)
		replicas[replicaName] = replica
	}

	primaryReplica := replicas[replicaNames[0]]
	primaryNQN := helpertypes.GetNQN(primaryReplica.Name)
	primaryPort := strconv.Itoa(int(primaryReplica.PortStart))

	_, err = helperinitiator.ConnectTarget(ip, primaryPort, primaryNQN, ne)
	c.Assert(err, IsNil)
	defer func() {
		err := helperinitiator.DisconnectTarget(primaryNQN, ne)
		c.Assert(err, IsNil)
	}()

	devices, err := helperinitiator.GetDevices(ip, primaryPort, primaryNQN, ne)
	c.Assert(err, IsNil)
	c.Assert(len(devices), Equals, 1)
	c.Assert(len(devices[0].Namespaces), Equals, 1)

	endpoint := filepath.Join("/dev", devices[0].Namespaces[0].NameSpace)
	err = writeDataToBlockDevice(ne, endpoint, 0, 8)
	c.Assert(err, IsNil)

	var primaryHeadSize uint64
	var secondaryHeadSize uint64
	for i := 0; i < 20; i++ {
		primaryReplicaInfo, err := spdkCli.ReplicaGet(replicaNames[0])
		c.Assert(err, IsNil)
		secondaryReplicaInfo, err := spdkCli.ReplicaGet(replicaNames[1])
		c.Assert(err, IsNil)

		primaryHeadSize = primaryReplicaInfo.Head.ActualSize
		secondaryHeadSize = secondaryReplicaInfo.Head.ActualSize
		if primaryHeadSize > secondaryHeadSize {
			break
		}
		time.Sleep(500 * time.Millisecond)
	}
	c.Assert(primaryHeadSize > secondaryHeadSize, Equals, true)

	replicaAddressMap := make(map[string]string)
	for _, replica := range replicas {
		replicaAddressMap[replica.Name] = net.JoinHostPort(ip, strconv.Itoa(int(replica.PortStart)))
	}

	engine, err := spdkCli.EngineCreate(engineName, volumeName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize, replicaAddressMap, 1, true)
	c.Assert(err, IsNil)
	c.Assert(engine, NotNil)
	c.Assert(engine.State, Equals, types.InstanceStateRunning)
	c.Assert(len(engine.ReplicaAddressMap), Equals, 1)
	c.Assert(engine.ReplicaAddressMap[replicaNames[0]], Equals, net.JoinHostPort(ip, strconv.Itoa(int(replicas[replicaNames[0]].PortStart))))
	c.Assert(engine.ReplicaModeMap[replicaNames[0]], Equals, types.ModeRW)

	// Create engine frontend and verify IO works on the salvaged engine
	e, err := spdkCli.EngineGet(engineName)
	c.Assert(err, IsNil)

	engineFrontend, err := spdkCli.EngineFrontendCreate(engineFrontendName, volumeName, engineName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize,
		net.JoinHostPort(e.IP, strconv.Itoa(int(e.Port))), 0, 0)
	c.Assert(err, IsNil)
	c.Assert(engineFrontend.State, Equals, types.InstanceStateRunning)

	salvageEndpoint := helperutil.GetLonghornDevicePath(volumeName)
	c.Assert(engineFrontend.Endpoint, Equals, salvageEndpoint)

	err = formatBlockDevice(salvageEndpoint, "ext4")
	c.Assert(err, IsNil)
}

func GetBlockDeviceSize(ne *commonns.Executor, endpoint string) int64 {
	output, err := ne.Execute(nil, "blockdev", []string{"--getsize64", endpoint}, 10*time.Second)
	if err != nil {
		return 0
	}
	size, err := strconv.ParseInt(strings.TrimSpace(output), 10, 64)
	if err != nil {
		return 0
	}
	return size
}

func (s *TestSuite) TestSPDKMultipleThread(c *C) {
	fmt.Println("Testing SPDK basic operations with multiple threads")

	diskDriverName := "aio"

	ip, err := commonnet.GetAnyExternalIP()
	c.Assert(err, IsNil)
	err = os.Setenv(commonnet.EnvPodIP, ip)
	c.Assert(err, IsNil)

	ctx, cancel := context.WithCancel(context.Background())
	var spdkWg sync.WaitGroup

	defer func() {
		cancel()
		spdkWg.Wait()
	}()

	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	c.Assert(err, IsNil)
	LaunchTestSPDKGRPCServer(ctx, c, ip, ne.Execute, &spdkWg)

	loopDevicePath := PrepareDiskFile(c)
	defer func() {
		CleanupDiskFile(c, loopDevicePath)
	}()

	spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
	c.Assert(err, IsNil)
	defer func() {
		if errClose := spdkCli.Close(); errClose != nil {
			logrus.WithError(errClose).Error("Failed to close SPDK client")
		}
	}()

	disk, err := spdkCli.DiskCreate(defaultTestDiskName, "", loopDevicePath, diskDriverName, int64(defaultTestBlockSize))
	c.Assert(err, IsNil)
	c.Assert(disk, NotNil)

	disk, err = waitForDiskReady(ctx, spdkCli, loopDevicePath, diskDriverName)
	c.Assert(err, IsNil)
	c.Assert(disk.Path, Equals, loopDevicePath)
	c.Assert(disk.Uuid, Not(Equals), "")

	defer func() {
		err := spdkCli.DiskDelete(defaultTestDiskName, disk.Uuid, disk.Path, diskDriverName)
		c.Assert(err, IsNil)

		disk, err = spdkCli.DiskGet(defaultTestDiskName, disk.Path, diskDriverName)
		c.Assert(err, NotNil)
		c.Assert(disk, IsNil)
	}()

	concurrentCount := 5
	dataCountInMB := int64(100)
	wg := sync.WaitGroup{}
	wg.Add(concurrentCount)
	for i := range concurrentCount {
		spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
		c.Assert(err, IsNil)
		defer func() {
			if errClose := spdkCli.Close(); errClose != nil {
				logrus.WithError(errClose).Error("Failed to close SPDK client")
			}
		}()

		volumeName := fmt.Sprintf("test-vol-%d", i)
		engineName := fmt.Sprintf("%s-e", volumeName)
		engineFrontendName := fmt.Sprintf("%s-ef", volumeName)
		replicaName1 := fmt.Sprintf("%s-replica-1", volumeName)
		replicaName2 := fmt.Sprintf("%s-replica-2", volumeName)
		replicaName3 := fmt.Sprintf("%s-replica-3", volumeName)

		go func() {
			defer func() {
				// Do cleanup
				err = spdkCli.EngineFrontendDelete(engineFrontendName)
				c.Assert(err, IsNil)
				err = spdkCli.EngineDelete(engineName)
				c.Assert(err, IsNil)
				err = spdkCli.ReplicaDelete(replicaName1, true)
				c.Assert(err, IsNil)
				err = spdkCli.ReplicaDelete(replicaName2, true)
				c.Assert(err, IsNil)
				err = spdkCli.ReplicaDelete(replicaName3, true)
				c.Assert(err, IsNil)

				wg.Done()
			}()

			replica1, err := spdkCli.ReplicaCreate(replicaName1, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
			c.Assert(err, IsNil)
			c.Assert(replica1.LvsName, Equals, defaultTestDiskName)
			c.Assert(replica1.LvsUUID, Equals, disk.Uuid)
			c.Assert(replica1.ErrorMsg, Equals, "")
			c.Assert(replica1.State, Equals, types.InstanceStateRunning)
			c.Assert(replica1.PortStart, Not(Equals), int32(0))
			c.Assert(replica1.Head, NotNil)
			c.Assert(replica1.Head.CreationTime, Not(Equals), "")
			c.Assert(replica1.Head.Parent, Equals, "")
			replica2, err := spdkCli.ReplicaCreate(replicaName2, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
			c.Assert(err, IsNil)
			c.Assert(replica2.LvsName, Equals, defaultTestDiskName)
			c.Assert(replica2.LvsUUID, Equals, disk.Uuid)
			c.Assert(replica2.ErrorMsg, Equals, "")
			c.Assert(replica2.State, Equals, types.InstanceStateRunning)
			c.Assert(replica2.PortStart, Not(Equals), int32(0))
			c.Assert(replica2.Head, NotNil)
			c.Assert(replica2.Head.CreationTime, Not(Equals), "")
			c.Assert(replica2.Head.Parent, Equals, "")

			replicaAddressMap := map[string]string{
				replica1.Name: net.JoinHostPort(ip, strconv.Itoa(int(replica1.PortStart))),
				replica2.Name: net.JoinHostPort(ip, strconv.Itoa(int(replica2.PortStart))),
			}
			replicaModeMap := map[string]types.Mode{
				replica1.Name: types.ModeRW,
				replica2.Name: types.ModeRW,
			}
			endpoint := helperutil.GetLonghornDevicePath(volumeName)
			engine, err := spdkCli.EngineCreate(engineName, volumeName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize, replicaAddressMap, 1, false)
			c.Assert(err, IsNil)
			c.Assert(engine.ErrorMsg, Equals, "")
			c.Assert(engine.State, Equals, types.InstanceStateRunning)
			c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
			c.Assert(engine.ReplicaModeMap, DeepEquals, replicaModeMap)

			engineFrontend, err := spdkCli.EngineFrontendCreate(engineFrontendName, volumeName, engineName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize,
				net.JoinHostPort(engine.IP, strconv.Itoa(int(engine.Port))), 0, 0)
			c.Assert(err, IsNil)
			c.Assert(engineFrontend.Endpoint, Equals, endpoint)
			c.Assert(engine.IP, Not(Equals), "")
			c.Assert(engine.Port, Not(Equals), int32(0))

			err = writeDataToBlockDevice(ne, endpoint, 0, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore1, err := util.GetFileChunkChecksum(endpoint, 0, 100*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore1, Not(Equals), "")

			snapshotName1 := "snap1"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName1)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName1, false)
			c.Assert(err, IsNil)
			for replicaName := range replicaAddressMap {
				waitReplicaSnapshotChecksum(c, spdkCli, replicaName, snapshotName1)
			}

			err = writeDataToBlockDevice(ne, endpoint, 200, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore2, err := util.GetFileChunkChecksum(endpoint, 200*helpertypes.MiB, 100*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore2, Not(Equals), "")

			snapshotName2 := "snap2"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName2)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName2, false)
			c.Assert(err, IsNil)

			// Check both replica snapshot map after the snapshot operations
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName1: {snapshotName2},
					snapshotName2: {types.VolumeHead},
				},
				map[string]api.SnapshotOptions{
					snapshotName1: {UserCreated: true},
					snapshotName2: {UserCreated: true},
				}, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			err = spdkCli.EngineFrontendSnapshotDelete(engineFrontendName, snapshotName1)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName2, false)
			c.Assert(err, IsNil)
			for replicaName := range replicaAddressMap {
				waitReplicaSnapshotChecksum(c, spdkCli, replicaName, snapshotName2)
			}

			// Detach and re-attach the volume
			err = spdkCli.EngineFrontendDelete(engineFrontendName)
			c.Assert(err, IsNil)
			err = spdkCli.EngineDelete(engineName)
			c.Assert(err, IsNil)
			err = spdkCli.ReplicaDelete(replicaName1, false)
			c.Assert(err, IsNil)
			err = spdkCli.ReplicaDelete(replicaName2, false)
			c.Assert(err, IsNil)

			replica1, err = spdkCli.ReplicaGet(replicaName1)
			c.Assert(err, IsNil)
			c.Assert(replica1.LvsName, Equals, defaultTestDiskName)
			c.Assert(replica1.LvsUUID, Equals, disk.Uuid)
			c.Assert(replica1.State, Equals, types.InstanceStateStopped)
			c.Assert(replica1.PortStart, Equals, int32(0))
			c.Assert(replica1.PortEnd, Equals, int32(0))

			replica2, err = spdkCli.ReplicaGet(replicaName1)
			c.Assert(err, IsNil)
			c.Assert(replica2.LvsName, Equals, defaultTestDiskName)
			c.Assert(replica2.LvsUUID, Equals, disk.Uuid)
			c.Assert(replica2.State, Equals, types.InstanceStateStopped)
			c.Assert(replica2.PortStart, Equals, int32(0))
			c.Assert(replica2.PortEnd, Equals, int32(0))

			replica1, err = spdkCli.ReplicaCreate(replicaName1, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
			c.Assert(err, IsNil)
			c.Assert(replica1.ErrorMsg, Equals, "")
			c.Assert(replica1.State, Equals, types.InstanceStateRunning)
			replica2, err = spdkCli.ReplicaCreate(replicaName2, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
			c.Assert(err, IsNil)
			c.Assert(replica2.ErrorMsg, Equals, "")
			c.Assert(replica2.State, Equals, types.InstanceStateRunning)

			replicaAddressMap = map[string]string{
				replica1.Name: net.JoinHostPort(ip, strconv.Itoa(int(replica1.PortStart))),
				replica2.Name: net.JoinHostPort(ip, strconv.Itoa(int(replica2.PortStart))),
			}
			engine, err = spdkCli.EngineCreate(engineName, volumeName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize, replicaAddressMap, 1, false)
			c.Assert(err, IsNil)
			c.Assert(engine.ErrorMsg, Equals, "")
			c.Assert(engine.State, Equals, types.InstanceStateRunning)
			c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
			c.Assert(engine.Port, Not(Equals), int32(0))

			engineFrontend, err = spdkCli.EngineFrontendCreate(engineFrontendName, volumeName, engineName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize,
				net.JoinHostPort(engine.IP, strconv.Itoa(int(engine.Port))), 0, 0)
			c.Assert(err, IsNil)
			c.Assert(engineFrontend.Endpoint, Equals, endpoint)

			// Check both replica snapshot map after the snapshot deletion and volume re-attachment
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName2: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Data keeps intact after the snapshot deletion and volume re-attachment
			cksumAfterSnap1, err := util.GetFileChunkChecksum(endpoint, 0, 100*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfterSnap1, Equals, cksumBefore1)
			cksumAfterSnap2, err := util.GetFileChunkChecksum(endpoint, 200*helpertypes.MiB, 100*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfterSnap2, Equals, cksumBefore2)

			// Before testing online rebuilding
			// Crash replica2 and remove it from the engine
			delete(replicaAddressMap, replicaName2)
			err = spdkCli.ReplicaDelete(replicaName2, true)
			c.Assert(err, IsNil)
			err = spdkCli.EngineReplicaDelete(engineName, replicaName2, net.JoinHostPort(ip, strconv.Itoa(int(replica2.PortStart))))
			c.Assert(err, IsNil)
			engine, err = spdkCli.EngineGet(engineName)
			c.Assert(err, IsNil)
			c.Assert(engine.ErrorMsg, Equals, "")
			c.Assert(engine.State, Equals, types.InstanceStateRunning)
			c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
			c.Assert(engine.ReplicaModeMap, DeepEquals, map[string]types.Mode{replicaName1: types.ModeRW})

			// Start testing online rebuilding
			// Launch a new replica then ask the engine to rebuild it
			replica3, err := spdkCli.ReplicaCreate(replicaName3, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
			c.Assert(err, IsNil)
			c.Assert(replica3.LvsName, Equals, defaultTestDiskName)
			c.Assert(replica3.LvsUUID, Equals, disk.Uuid)
			c.Assert(replica3.ErrorMsg, Equals, "")
			c.Assert(replica3.State, Equals, types.InstanceStateRunning)
			c.Assert(replica3.PortStart, Not(Equals), int32(0))
			c.Assert(replica3.Head, NotNil)
			c.Assert(replica3.Head.CreationTime, Not(Equals), "")
			c.Assert(replica3.Head.Parent, Equals, "")

			err = spdkCli.EngineFrontendReplicaAdd(engineFrontendName, replicaName3, net.JoinHostPort(ip, strconv.Itoa(int(replica3.PortStart))), defaultTestFastSync)
			c.Assert(err, IsNil)

			WaitForReplicaRebuildingComplete(c, spdkCli, engineName, replicaName3)

			snapshotNameRebuild := ""
			for replicaName := range replicaAddressMap {
				replica, err := spdkCli.ReplicaGet(replicaName)
				c.Assert(err, IsNil)
				for snapName, snapLvol := range replica.Snapshots {
					if strings.HasPrefix(snapName, server.RebuildingSnapshotNamePrefix) {
						c.Assert(snapLvol.Children[types.VolumeHead], Equals, true)
						if snapshotNameRebuild == "" {
							snapshotNameRebuild = snapName
						} else {
							c.Assert(snapName, Equals, snapshotNameRebuild)
						}
						break
					}
				}
			}
			c.Assert(snapshotNameRebuild, Not(Equals), "")

			err = spdkCli.EngineSnapshotHash(engineName, snapshotName2, false)
			c.Assert(err, IsNil)

			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName3},
				map[string][]string{
					snapshotName2:       {snapshotNameRebuild},
					snapshotNameRebuild: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Verify the rebuilding result
			replicaAddressMap = map[string]string{
				replica1.Name: net.JoinHostPort(ip, strconv.Itoa(int(replica1.PortStart))),
				replica3.Name: net.JoinHostPort(ip, strconv.Itoa(int(replica3.PortStart))),
			}
			engine, err = spdkCli.EngineGet(engineName)
			c.Assert(err, IsNil)
			c.Assert(engine.ErrorMsg, Equals, "")
			c.Assert(engine.State, Equals, types.InstanceStateRunning)
			c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
			c.Assert(engine.ReplicaModeMap, DeepEquals, map[string]types.Mode{replicaName1: types.ModeRW, replicaName3: types.ModeRW})

			// The newly rebuilt replica should contain correct data
			cksumAfterRebuilding1, err := util.GetFileChunkChecksum(endpoint, 0, 100*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfterRebuilding1, Equals, cksumBefore1)
			cksumAfterRebuilding2, err := util.GetFileChunkChecksum(endpoint, 200*helpertypes.MiB, 100*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfterRebuilding2, Equals, cksumBefore2)

			for _, replicaName := range []string{replicaName1, replicaName3} {
				waitReplicaSnapshotChecksum(c, spdkCli, replicaName, "")
			}
		}()
	}

	wg.Wait()

	engineFrontendList, err := spdkCli.EngineFrontendList()
	c.Assert(err, IsNil)
	for _, ef := range engineFrontendList {
		err = spdkCli.EngineFrontendDelete(ef.Name)
		c.Assert(err, IsNil)
	}

	engineList, err := spdkCli.EngineList()
	c.Assert(err, IsNil)
	for _, engine := range engineList {
		c.Assert(err, IsNil)

		err = spdkCli.EngineDelete(engine.Name)
		c.Assert(err, IsNil)
	}
	replicaList, err := spdkCli.ReplicaList()
	c.Assert(err, IsNil)
	for _, replica := range replicaList {
		err = spdkCli.ReplicaDelete(replica.Name, true)
		c.Assert(err, IsNil)
	}
}

func (s *TestSuite) spdkMultipleThreadSnapshotOpsAndRebuilding(c *C, withBackingImage bool) {
	diskDriverName := "aio"

	ip, err := commonnet.GetAnyExternalIP()
	c.Assert(err, IsNil)
	err = os.Setenv(commonnet.EnvPodIP, ip)
	c.Assert(err, IsNil)

	ctx, cancel := context.WithCancel(context.Background())
	var spdkWg sync.WaitGroup

	defer func() {
		cancel()
		spdkWg.Wait()
	}()

	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	c.Assert(err, IsNil)
	LaunchTestSPDKGRPCServer(ctx, c, ip, ne.Execute, &spdkWg)

	loopDevicePath := PrepareDiskFile(c)
	defer func() {
		CleanupDiskFile(c, loopDevicePath)
	}()

	spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
	c.Assert(err, IsNil)
	defer func() {
		if errClose := spdkCli.Close(); errClose != nil {
			logrus.WithError(errClose).Error("Failed to close SPDK client")
		}
	}()

	disk, err := spdkCli.DiskCreate(defaultTestDiskName, "", loopDevicePath, diskDriverName, int64(defaultTestBlockSize))
	c.Assert(err, IsNil)
	c.Assert(disk, NotNil)

	disk, err = waitForDiskReady(ctx, spdkCli, loopDevicePath, diskDriverName)
	c.Assert(err, IsNil)
	c.Assert(disk.Path, Equals, loopDevicePath)
	c.Assert(disk.Uuid, Not(Equals), "")

	defer func() {
		err := spdkCli.DiskDelete(defaultTestDiskName, disk.Uuid, disk.Path, diskDriverName)
		c.Assert(err, IsNil)

		disk, err = spdkCli.DiskGet(defaultTestDiskName, disk.Path, diskDriverName)
		c.Assert(err, NotNil)
		c.Assert(disk, IsNil)
	}()

	var bi *api.BackingImage
	if withBackingImage {
		bi, err = spdkCli.BackingImageCreate(defaultTestBackingImageName, defaultTestBackingImageUUID, disk.Uuid, defaultTestBackingImageSize, defaultTestBackingImageChecksum, defaultTestBackingImageDownloadURL, "")
		c.Assert(err, IsNil)
		c.Assert(bi, NotNil)
		defer func() {
			err := spdkCli.BackingImageDelete(defaultTestBackingImageName, disk.Uuid)
			c.Assert(err, IsNil)
		}()

		// check if bi.State is "ready" in 300 seconds
		for i := range maxBackingImageGetRetries {
			bi, err = spdkCli.BackingImageGet(defaultTestBackingImageName, disk.Uuid)
			c.Assert(err, IsNil)

			if bi.State == string(types.BackingImageStateReady) {
				break
			}

			time.Sleep(1 * time.Second)

			if i == maxBackingImageGetRetries-1 {
				c.Assert(bi.State, Equals, string(types.BackingImageStateReady))
			}
		}
	}

	concurrentCount := 5
	dataCountInMB := int64(10)
	wg := sync.WaitGroup{}
	wg.Add(concurrentCount)
	for i := range concurrentCount {
		spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
		c.Assert(err, IsNil)
		defer func() {
			if errClose := spdkCli.Close(); errClose != nil {
				logrus.WithError(errClose).Error("Failed to close SPDK client")
			}
		}()

		volumeName := fmt.Sprintf("test-vol-%d", i)
		engineName := fmt.Sprintf("%s-e", volumeName)
		engineFrontendName := fmt.Sprintf("%s-ef", volumeName)
		replicaName1 := fmt.Sprintf("%s-replica-1", volumeName)
		replicaName2 := fmt.Sprintf("%s-replica-2", volumeName)
		replicaName3 := fmt.Sprintf("%s-replica-3", volumeName)
		replicaName4 := fmt.Sprintf("%s-replica-4", volumeName)

		go func() {
			defer func() {
				// Do cleanup
				err = spdkCli.EngineFrontendDelete(engineFrontendName)
				c.Assert(err, IsNil)
				err = spdkCli.EngineDelete(engineName)
				c.Assert(err, IsNil)
				err = spdkCli.ReplicaDelete(replicaName1, true)
				c.Assert(err, IsNil)
				err = spdkCli.ReplicaDelete(replicaName2, true)
				c.Assert(err, IsNil)
				err = spdkCli.ReplicaDelete(replicaName3, true)
				c.Assert(err, IsNil)

				wg.Done()
			}()

			backingImageName := ""
			if withBackingImage {
				backingImageName = bi.Name
			}
			replica1, err := spdkCli.ReplicaCreate(replicaName1, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, backingImageName)

			c.Assert(err, IsNil)
			c.Assert(replica1.LvsName, Equals, defaultTestDiskName)
			c.Assert(replica1.LvsUUID, Equals, disk.Uuid)
			c.Assert(replica1.ErrorMsg, Equals, "")
			c.Assert(replica1.State, Equals, types.InstanceStateRunning)
			c.Assert(replica1.PortStart, Not(Equals), int32(0))
			replica2, err := spdkCli.ReplicaCreate(replicaName2, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, backingImageName)
			c.Assert(err, IsNil)
			c.Assert(replica2.LvsName, Equals, defaultTestDiskName)
			c.Assert(replica2.LvsUUID, Equals, disk.Uuid)
			c.Assert(replica2.ErrorMsg, Equals, "")
			c.Assert(replica2.State, Equals, types.InstanceStateRunning)
			c.Assert(replica2.PortStart, Not(Equals), int32(0))

			_, err = spdkCli.ReplicaGet(replicaName1)
			c.Assert(err, IsNil)
			_, err = spdkCli.ReplicaGet(replicaName2)
			c.Assert(err, IsNil)

			replicaAddressMap := map[string]string{
				replica1.Name: net.JoinHostPort(ip, strconv.Itoa(int(replica1.PortStart))),
				replica2.Name: net.JoinHostPort(ip, strconv.Itoa(int(replica2.PortStart))),
			}
			replicaModeMap := map[string]types.Mode{
				replica1.Name: types.ModeRW,
				replica2.Name: types.ModeRW,
			}
			endpoint := helperutil.GetLonghornDevicePath(volumeName)
			engine, err := spdkCli.EngineCreate(engineName, volumeName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize, replicaAddressMap, 1, false)
			c.Assert(err, IsNil)
			c.Assert(engine.ErrorMsg, Equals, "")
			c.Assert(engine.State, Equals, types.InstanceStateRunning)
			c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
			c.Assert(engine.ReplicaModeMap, DeepEquals, replicaModeMap)
			c.Assert(engine.Port, Not(Equals), int32(0))
			c.Assert(engine.Port, Not(Equals), int32(0))

			engineFrontend, err := spdkCli.EngineFrontendCreate(engineFrontendName, volumeName, engineName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize,
				net.JoinHostPort(engine.IP, strconv.Itoa(int(engine.Port))), 0, 0)
			c.Assert(err, IsNil)
			c.Assert(engineFrontend.Endpoint, Equals, endpoint)

			offsetInMB := int64(0)
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore11, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore11, Not(Equals), "")
			snapshotName11 := "snap11"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName11)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName11, false)
			c.Assert(err, IsNil)

			// Current snapshot tree (with backing image):
			//       nil (backing image) -> snap11 -> head
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore12, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore12, Not(Equals), "")
			snapshotName12 := "snap12"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName12)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName12, false)
			c.Assert(err, IsNil)

			// Current snapshot tree (with backing image):
			//       nil (backing image) -> snap11 -> snap12 -> head
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12},
					snapshotName12: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = 2 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore13, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore13, Not(Equals), "")
			snapshotName13 := "snap13"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName13)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName13, false)
			c.Assert(err, IsNil)

			// Current snapshot tree (with backing image):
			//       nil (backing image) -> snap11 -> snap12 -> snap13 -> head
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12},
					snapshotName12: {snapshotName13},
					snapshotName13: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = 3 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore14, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore14, Not(Equals), "")
			snapshotName14 := "snap14"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName14)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName14, false)
			c.Assert(err, IsNil)

			// Current snapshot tree (with backing image):
			//       nil (backing image) -> snap11 -> snap12 -> snap13 -> snap14 -> head
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12},
					snapshotName12: {snapshotName13},
					snapshotName13: {snapshotName14},
					snapshotName14: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = 4 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore15, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore15, Not(Equals), "")
			snapshotName15 := "snap15"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName15)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName15, false)
			c.Assert(err, IsNil)

			// Current snapshot tree (with backing image):
			//       nil (backing image) -> snap11 -> snap12 -> snap13 -> snap14 -> snap15 -> head
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12},
					snapshotName12: {snapshotName13},
					snapshotName13: {snapshotName14},
					snapshotName14: {snapshotName15},
					snapshotName15: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Current snapshot tree (with backing image):
			// 	 nil (backing image) -> snap11[0,10] -> snap12[10,20] -> snap13[20,30] -> snap14[30,40] -> snap15[40,50] -> head[50,60]

			// Write some extra data into the current head before reverting. This part of data will be discarded after revert
			offsetInMB = 5 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore16, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)

			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12},
					snapshotName12: {snapshotName13},
					snapshotName13: {snapshotName14},
					snapshotName14: {snapshotName15},
					snapshotName15: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Revert for a new chain (chain 2)
			revertSnapshot(c, spdkCli, snapshotName13, volumeName, engineName, engineFrontendName, replicaAddressMap)

			// Current snapshot tree (with backing image):
			// 	 nil (backing image) -> snap11[0,10] -> snap12[10,20] -> snap13[20,30] -> snap14[30,40] -> snap15[40,50]
			//                                                             \
			//                                                               -> head
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12},
					snapshotName12: {snapshotName13},
					snapshotName13: {snapshotName14, types.VolumeHead},
					snapshotName14: {snapshotName15},
					snapshotName15: {},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Only the data of snap11, snap12, and snap13 keeps intact after the snapshot deletion and volume re-attachment
			offsetInMB = 0
			cksumAfter11, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter11, Equals, cksumBefore11)
			offsetInMB = dataCountInMB
			cksumAfter12, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter12, Equals, cksumBefore12)
			offsetInMB = 2 * dataCountInMB
			cksumAfter13, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter13, Equals, cksumBefore13)
			// The data of snap14 is no longer there after reverting to snap13
			offsetInMB = 3 * dataCountInMB
			cksumAfter14, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter14, Not(Equals), cksumBefore14)

			offsetInMB = 3 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore21, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore21, Not(Equals), "")
			snapshotName21 := "snap21"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName21)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName21, false)
			c.Assert(err, IsNil)

			// Current snapshot tree (with backing image):
			// 	 nil (backing image) -> snap11[0,10] -> snap12[10,20] -> snap13[20,30] -> snap14[30,40] -> snap15[40,50]
			//                                                             \
			//                                                               -> snap21 ->  head
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12},
					snapshotName12: {snapshotName13},
					snapshotName13: {snapshotName14, snapshotName21},
					snapshotName21: {types.VolumeHead},
					snapshotName14: {snapshotName15},
					snapshotName15: {},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = 4 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore22, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore22, Not(Equals), "")
			snapshotName22 := "snap22"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName22)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName22, false)
			c.Assert(err, IsNil)

			// Current snapshot tree (with backing image):
			// 	 nil (backing image) -> snap11[0,10] -> snap12[10,20] -> snap13[20,30] -> snap14[30,40] -> snap15[40,50]
			//                                                             \
			//                                                               -> snap21 -> snap22 -> head
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12},
					snapshotName12: {snapshotName13},
					snapshotName13: {snapshotName14, snapshotName21},
					snapshotName21: {snapshotName22},
					snapshotName22: {types.VolumeHead},
					snapshotName14: {snapshotName15},
					snapshotName15: {},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = 5 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore23, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore23, Not(Equals), "")
			snapshotName23 := "snap23"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName23)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName23, false)
			c.Assert(err, IsNil)

			// Current snapshot tree (with backing image):
			// 	 nil (backing image) -> snap11[0,10] -> snap12[10,20] -> snap13[20,30] -> snap14[30,40] -> snap15[40,50]
			// 	                                                                       \
			// 	                                                                        -> snap21[30,40] -> snap22[40,50] -> snap23[50,60] -> head[60,60]

			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12},
					snapshotName12: {snapshotName13},
					snapshotName13: {snapshotName14, snapshotName21},
					snapshotName14: {snapshotName15},
					snapshotName15: {},
					snapshotName21: {snapshotName22},
					snapshotName22: {snapshotName23},
					snapshotName23: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Delete some snapshots
			err = spdkCli.EngineFrontendSnapshotDelete(engineFrontendName, snapshotName21)
			c.Assert(err, IsNil)
			err = spdkCli.EngineFrontendSnapshotDelete(engineFrontendName, snapshotName22)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName23, false)
			c.Assert(err, IsNil)
			for _, replicaName := range []string{replicaName1, replicaName2} {
				waitReplicaSnapshotChecksum(c, spdkCli, replicaName, snapshotName23)
			}

			err = spdkCli.EngineFrontendSnapshotDelete(engineFrontendName, snapshotName12)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName13, false)
			c.Assert(err, IsNil)
			for _, replicaName := range []string{replicaName1, replicaName2} {
				waitReplicaSnapshotChecksum(c, spdkCli, replicaName, snapshotName13)
			}
			err = spdkCli.EngineFrontendSnapshotDelete(engineFrontendName, snapshotName14)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName15, false)
			c.Assert(err, IsNil)
			for _, replicaName := range []string{replicaName1, replicaName2} {
				waitReplicaSnapshotChecksum(c, spdkCli, replicaName, snapshotName15)
			}
			err = spdkCli.EngineFrontendSnapshotDelete(engineFrontendName, snapshotName13)
			c.Assert(strings.Contains(err.Error(), "since it contains multiple children"), Equals, true)

			// Current snapshot tree (with backing image):
			// 	 nil (backing image) -> snap11[0,10] -> snap13[10,30] -> snap15[30,50]
			// 	                                                      \
			// 	                                                       -> snap23[30,60] -> head[60,60]

			// Verify the data for the current snapshot chain (chain 2)
			offsetInMB = 0
			cksumAfter11, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter11, Equals, cksumBefore11)
			offsetInMB = dataCountInMB
			cksumAfter12, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter12, Equals, cksumBefore12)
			offsetInMB = 2 * dataCountInMB
			cksumAfter13, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter13, Equals, cksumBefore13)

			offsetInMB = 3 * dataCountInMB
			cksumAfter21, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter21, Equals, cksumBefore21)
			offsetInMB = 4 * dataCountInMB
			cksumAfter22, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter22, Equals, cksumBefore22)
			offsetInMB = 5 * dataCountInMB
			cksumAfter23, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter23, Equals, cksumBefore23)

			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName13},
					snapshotName13: {snapshotName15, snapshotName23},
					snapshotName15: {},
					snapshotName23: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// TODO: Add replica rebuilding related test step

			// Revert for a new chain (chain 3)
			revertSnapshot(c, spdkCli, snapshotName11, volumeName, engineName, engineFrontendName, replicaAddressMap)

			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName13},
					snapshotName13: {snapshotName15, snapshotName23},
					snapshotName15: {},
					snapshotName23: {},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Create and delete some snapshots for the new chain (chain 3)
			offsetInMB = dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore31, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore31, Not(Equals), "")
			snapshotName31 := "snap31"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName31)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName31, false)
			c.Assert(err, IsNil)

			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName13, snapshotName31},
					snapshotName13: {snapshotName15, snapshotName23},
					snapshotName15: {},
					snapshotName23: {},
					snapshotName31: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = 2 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore32, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore32, Not(Equals), "")
			snapshotName32 := "snap32"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName32)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName32, false)
			c.Assert(err, IsNil)

			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName13, snapshotName31},
					snapshotName13: {snapshotName15, snapshotName23},
					snapshotName15: {},
					snapshotName23: {},
					snapshotName31: {snapshotName32},
					snapshotName32: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			err = spdkCli.EngineFrontendSnapshotDelete(engineFrontendName, snapshotName31)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName32, false)
			c.Assert(err, IsNil)

			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName13, snapshotName32},
					snapshotName13: {snapshotName15, snapshotName23},
					snapshotName15: {},
					snapshotName23: {},
					snapshotName32: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Current snapshot tree (with backing image):
			// 	 nil (backing image) -> snap11[0,10] -> snap13[10,30] -> snap15[30,50]
			// 	                                     \                \
			// 	                                      \                -> snap23[30,60]
			// 	                                       \
			// 	                                        -> snap32[10,30] -> head[30,30]

			offsetInMB = 0
			cksumAfter11, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter11, Equals, cksumBefore11)

			offsetInMB = dataCountInMB
			cksumAfter31, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter31, Equals, cksumBefore31)
			offsetInMB = 2 * dataCountInMB
			cksumAfter32, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter32, Equals, cksumBefore32)

			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName13, snapshotName32},
					snapshotName13: {snapshotName15, snapshotName23},
					snapshotName15: {},
					snapshotName23: {},
					snapshotName32: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Revert for a new chain (chain 4)
			revertSnapshot(c, spdkCli, snapshotName11, volumeName, engineName, engineFrontendName, replicaAddressMap)

			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName13, snapshotName32, types.VolumeHead},
					snapshotName13: {snapshotName15, snapshotName23},
					snapshotName15: {},
					snapshotName23: {},
					snapshotName32: {},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Create some snapshots for the new chain (chain 4)
			offsetInMB = dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore41, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore41, Not(Equals), "")
			snapshotName41 := "snap41"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName41)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName41, false)
			c.Assert(err, IsNil)

			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName13, snapshotName32, snapshotName41},
					snapshotName13: {snapshotName15, snapshotName23},
					snapshotName15: {},
					snapshotName23: {},
					snapshotName32: {},
					snapshotName41: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = 2 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore42, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore42, Not(Equals), "")
			snapshotName42 := "snap42"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName42)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName42, false)
			c.Assert(err, IsNil)

			// Current snapshot tree (with backing image):
			// 	 nil (backing image) -> snap11[0,10] -> snap13[10,30] -> snap15[30,50]
			// 	                                    |\                \
			// 	                                    | \                -> snap23[30,60]
			// 	                                    |  \
			// 	                                    \   -> snap32[10,30]
			// 	                                     \
			// 	                                      -> snap41[10,20] -> snap42[20,30] -> head[30,30]

			offsetInMB = 0
			cksumAfter11, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter11, Equals, cksumBefore11)

			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName13, snapshotName32, snapshotName41},
					snapshotName13: {snapshotName15, snapshotName23},
					snapshotName15: {},
					snapshotName23: {},
					snapshotName32: {},
					snapshotName41: {snapshotName42},
					snapshotName42: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Revert back to chain 1
			revertSnapshot(c, spdkCli, snapshotName15, volumeName, engineName, engineFrontendName, replicaAddressMap)

			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName13, snapshotName32, snapshotName41},
					snapshotName13: {snapshotName15, snapshotName23},
					snapshotName15: {types.VolumeHead},
					snapshotName23: {},
					snapshotName32: {},
					snapshotName41: {snapshotName42},
					snapshotName42: {},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Test online rebuilding twice

			// Write some data to the head before the 1st rebuilding
			offsetInMB = 6 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBeforeRebuild11, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBeforeRebuild11, Not(Equals), "")

			// Crash replica1
			delete(replicaAddressMap, replicaName1)
			err = spdkCli.ReplicaDelete(replicaName1, true)
			c.Assert(err, IsNil)
			err = spdkCli.EngineReplicaDelete(engineName, replicaName1, net.JoinHostPort(ip, strconv.Itoa(int(replica1.PortStart))))
			c.Assert(err, IsNil)
			engine, err = spdkCli.EngineGet(engineName)
			c.Assert(err, IsNil)
			c.Assert(engine.State, Equals, types.InstanceStateRunning)

			c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
			c.Assert(engine.ReplicaModeMap, DeepEquals, map[string]types.Mode{replicaName2: types.ModeRW})
			// Launch the 1st rebuilding replica as the replacement of the crashed replica1
			replica3, err := spdkCli.ReplicaCreate(replicaName3, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, backingImageName)
			c.Assert(err, IsNil)
			c.Assert(replica3.LvsName, Equals, defaultTestDiskName)
			c.Assert(replica3.LvsUUID, Equals, disk.Uuid)
			c.Assert(replica3.State, Equals, types.InstanceStateRunning)
			c.Assert(replica3.PortStart, Not(Equals), int32(0))
			// Start the 1st rebuilding and wait for the completion
			err = spdkCli.EngineFrontendReplicaAdd(engineFrontendName, replicaName3, net.JoinHostPort(ip, strconv.Itoa(int(replica3.PortStart))), defaultTestFastSync)
			c.Assert(err, IsNil)
			WaitForReplicaRebuildingComplete(c, spdkCli, engineName, replicaName3)
			// While the volume head data written before rebuilding remains
			offsetInMB = 6 * dataCountInMB
			cksumAfterRebuild11, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfterRebuild11, Equals, cksumBeforeRebuild11)
			// Figure out the 1st rebuilding snapshot name
			replicaAddressMap[replica3.Name] = net.JoinHostPort(ip, strconv.Itoa(int(replica3.PortStart)))
			snapshotNameRebuild1 := ""
			for replicaName := range replicaAddressMap {
				replica, err := spdkCli.ReplicaGet(replicaName)
				c.Assert(err, IsNil)
				for snapName, snapLvol := range replica.Snapshots {
					if strings.HasPrefix(snapName, server.RebuildingSnapshotNamePrefix) {
						c.Assert(snapLvol.Children[types.VolumeHead], Equals, true)
						c.Assert(snapLvol.Parent, Equals, snapshotName15)
						if snapshotNameRebuild1 == "" {
							snapshotNameRebuild1 = snapName
						} else {
							c.Assert(snapName, Equals, snapshotNameRebuild1)
						}
						break
					}
				}
			}
			c.Assert(snapshotNameRebuild1, Not(Equals), "")

			// Verify the 1st rebuilding result
			engine, err = spdkCli.EngineGet(engineName)
			c.Assert(err, IsNil)
			c.Assert(engine.State, Equals, types.InstanceStateRunning)

			c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
			c.Assert(engine.ReplicaModeMap, DeepEquals, map[string]types.Mode{replicaName2: types.ModeRW, replicaName3: types.ModeRW})

			// Rebuilding once leads to 1 snapshot creations (with random name)
			// Current snapshot tree (with backing image):
			// 	 nil (backing image) -> snap11[0,10] -> snap13[10,30] -> snap15[30,50] -> rebuild11[60,70] -> head[70,70]
			// 	                                    |\                \
			// 	                                    | \                -> snap23[30,60]
			// 	                                    |  \
			// 	                                    \   -> snap32[10,30]
			// 	                                     \
			// 	                                      -> snap41[10,20] -> snap42[20,30]

			snapshotMap := map[string][]string{
				snapshotName11:       {snapshotName13, snapshotName32, snapshotName41},
				snapshotName13:       {snapshotName15, snapshotName23},
				snapshotName15:       {snapshotNameRebuild1},
				snapshotNameRebuild1: {types.VolumeHead},
				snapshotName23:       {},
				snapshotName32:       {},
				snapshotName41:       {snapshotName42},
				snapshotName42:       {},
			}
			snapshotOpts := map[string]api.SnapshotOptions{}
			for snapName := range snapshotMap {
				snapshotOpts[snapName] = api.SnapshotOptions{UserCreated: true}
			}
			snapshotOpts[snapshotNameRebuild1] = api.SnapshotOptions{UserCreated: false}

			for snapName := range snapshotMap {
				err = spdkCli.EngineSnapshotHash(engineName, snapName, false)
				c.Assert(err, IsNil)
			}

			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName2, replicaName3}, snapshotMap, snapshotOpts, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Write more data to the head before the 2nd rebuilding
			offsetInMB = 7 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBeforeRebuild12, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBeforeRebuild12, Not(Equals), "")

			// Crash replica2
			delete(replicaAddressMap, replicaName2)
			err = spdkCli.ReplicaDelete(replicaName2, true)
			c.Assert(err, IsNil)
			err = spdkCli.EngineReplicaDelete(engineName, replicaName2, net.JoinHostPort(ip, strconv.Itoa(int(replica2.PortStart))))
			c.Assert(err, IsNil)
			engine, err = spdkCli.EngineGet(engineName)
			c.Assert(err, IsNil)
			c.Assert(engine.State, Equals, types.InstanceStateRunning)

			c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
			c.Assert(engine.ReplicaModeMap, DeepEquals, map[string]types.Mode{replicaName3: types.ModeRW})
			// Launch the 2nd rebuilding replica as the replacement of the crashed replica2
			replica4, err := spdkCli.ReplicaCreate(replicaName4, defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, backingImageName)
			c.Assert(err, IsNil)
			c.Assert(replica4.LvsName, Equals, defaultTestDiskName)
			c.Assert(replica4.LvsUUID, Equals, disk.Uuid)
			c.Assert(replica4.State, Equals, types.InstanceStateRunning)
			c.Assert(replica4.PortStart, Not(Equals), int32(0))
			// Start the 2nd rebuilding and wait for the completion
			err = spdkCli.EngineFrontendReplicaAdd(engineFrontendName, replicaName4, net.JoinHostPort(ip, strconv.Itoa(int(replica4.PortStart))), defaultTestFastSync)
			c.Assert(err, IsNil)
			WaitForReplicaRebuildingComplete(c, spdkCli, engineName, replicaName4)
			// While the volume head data written before rebuilding remains
			offsetInMB = 6 * dataCountInMB
			cksumAfterRebuild11, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfterRebuild11, Equals, cksumBeforeRebuild11)
			offsetInMB = 7 * dataCountInMB
			cksumAfterRebuild12, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfterRebuild12, Equals, cksumBeforeRebuild12)
			// Figure out the 2nd rebuilding snapshot name
			replicaAddressMap[replica4.Name] = net.JoinHostPort(ip, strconv.Itoa(int(replica4.PortStart)))
			snapshotNameRebuild2 := ""
			for replicaName := range replicaAddressMap {
				replica, err := spdkCli.ReplicaGet(replicaName)
				c.Assert(err, IsNil)
				for snapName, snapLvol := range replica.Snapshots {
					if snapName != snapshotNameRebuild1 && strings.HasPrefix(snapName, server.RebuildingSnapshotNamePrefix) {
						c.Assert(snapLvol.Children[types.VolumeHead], Equals, true)
						c.Assert(snapLvol.Parent, Equals, snapshotNameRebuild1)
						if snapshotNameRebuild2 == "" {
							snapshotNameRebuild2 = snapName
						} else {
							c.Assert(snapName, Equals, snapshotNameRebuild2)
						}
						break
					}
				}
			}
			c.Assert(snapshotNameRebuild2, Not(Equals), "")

			// Verify the 2nd rebuilding result
			engine, err = spdkCli.EngineGet(engineName)
			c.Assert(err, IsNil)
			c.Assert(engine.State, Equals, types.InstanceStateRunning)

			c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
			c.Assert(engine.ReplicaModeMap, DeepEquals, map[string]types.Mode{replicaName3: types.ModeRW, replicaName4: types.ModeRW})

			// Rebuilding twice leads to 2 snapshot creations (with random name)
			// Current snapshot tree (with backing image):
			// 	 nil (backing image) -> snap11[0,10] -> snap13[10,30] -> snap15[30,50] -> rebuild11[60,70] -> rebuild12[70,80] -> head[80,80]
			// 	                                    |\                \
			// 	                                    | \                -> snap23[30,60]
			// 	                                    |  \
			// 	                                    \   -> snap32[10,30]
			// 	                                     \
			// 	                                      -> snap41[10,20] -> snap42[20,30]
			snapshotMap = map[string][]string{
				snapshotName11:       {snapshotName13, snapshotName32, snapshotName41},
				snapshotName13:       {snapshotName15, snapshotName23},
				snapshotName15:       {snapshotNameRebuild1},
				snapshotNameRebuild1: {snapshotNameRebuild2},
				snapshotNameRebuild2: {types.VolumeHead},
				snapshotName23:       {},
				snapshotName32:       {},
				snapshotName41:       {snapshotName42},
				snapshotName42:       {},
			}
			snapshotOpts[snapshotNameRebuild2] = api.SnapshotOptions{UserCreated: false}

			for snapName := range snapshotMap {
				err = spdkCli.EngineSnapshotHash(engineName, snapName, false)
				c.Assert(err, IsNil)
			}

			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName3, replicaName4}, snapshotMap, snapshotOpts, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Purging snapshots would lead to rebuild11 and rebuild12 cleanup
			err = spdkCli.EngineFrontendSnapshotPurge(engineFrontendName)
			c.Assert(err, IsNil)

			// Current snapshot tree (with backing image):
			// nil (backing image) -> snap11[0,10] -> snap13[10,30] -> snap15[30,50] -> head[60,80]
			// 	                                    |\                \
			// 	                                    | \                -> snap23[30,60]
			// 	                                    |  \
			// 	                                    \   -> snap32[10,30]
			// 	                                     \
			// 	                                      -> snap41[10,20] -> snap42[20,30]
			snapshotMap = map[string][]string{
				snapshotName11: {snapshotName13, snapshotName32, snapshotName41},
				snapshotName13: {snapshotName15, snapshotName23},
				snapshotName15: {types.VolumeHead},
				snapshotName23: {},
				snapshotName32: {},
				snapshotName41: {snapshotName42},
				snapshotName42: {},
			}
			delete(snapshotOpts, snapshotNameRebuild1)
			delete(snapshotOpts, snapshotNameRebuild2)
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName3, replicaName4}, snapshotMap, snapshotOpts, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)
			for _, replicaName := range []string{replicaName3, replicaName4} {
				replica, err := spdkCli.ReplicaGet(replicaName)
				c.Assert(err, IsNil)
				c.Assert(replica.Head.Parent, Equals, snapshotName15)
				c.Assert(replica.Head.ActualSize, Equals, uint64(2*dataCountInMB*helpertypes.MiB))
			}

			// The newly rebuilt replicas should contain correct/unchanged data
			// Verify chain1
			offsetInMB = 0
			cksumAfter11, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter11, Equals, cksumBefore11)
			offsetInMB = dataCountInMB
			cksumAfter12, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter12, Equals, cksumBefore12)
			offsetInMB = 2 * dataCountInMB
			cksumAfter13, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter13, Equals, cksumBefore13)
			offsetInMB = 3 * dataCountInMB
			cksumAfter14, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter14, Equals, cksumBefore14)
			offsetInMB = 4 * dataCountInMB
			cksumAfter15, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter15, Equals, cksumBefore15)
			// Notice that the head before the first revert is discarded
			offsetInMB = 5 * dataCountInMB
			cksumAfter16, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter16, Not(Equals), cksumBefore16)
			// While the volume head data after rebuilding and purge still remains
			offsetInMB = 6 * dataCountInMB
			cksumAfterRebuild11, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfterRebuild11, Equals, cksumBeforeRebuild11)
			offsetInMB = 7 * dataCountInMB
			cksumAfterRebuild12, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfterRebuild12, Equals, cksumBeforeRebuild12)
			// Verify chain2
			revertSnapshot(c, spdkCli, snapshotName23, volumeName, engineName, engineFrontendName, replicaAddressMap)

			// Current snapshot tree (with backing image):
			// 	 nil (backing image) -> snap11[0,10] -> snap13[10,30] -> snap15[30,50]
			// 	                                    |\                \
			// 	                                    | \                -> snap23[30,60] -> head
			// 	                                    |  \
			// 	                                    \   -> snap32[10,30]
			// 	                                     \
			// 	                                      -> snap41[10,20] -> snap42[20,30]
			snapshotMap = map[string][]string{
				snapshotName11: {snapshotName13, snapshotName32, snapshotName41},
				snapshotName13: {snapshotName15, snapshotName23},
				snapshotName15: {},
				snapshotName23: {types.VolumeHead},
				snapshotName32: {},
				snapshotName41: {snapshotName42},
				snapshotName42: {},
			}
			delete(snapshotOpts, snapshotNameRebuild1)
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName3, replicaName4}, snapshotMap, snapshotOpts, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = 0
			cksumAfter11, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter11, Equals, cksumBefore11)
			offsetInMB = dataCountInMB
			cksumAfter12, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter12, Equals, cksumBefore12)
			offsetInMB = 2 * dataCountInMB
			cksumAfter13, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter13, Equals, cksumBefore13)
			offsetInMB = 3 * dataCountInMB
			cksumAfter21, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter21, Equals, cksumBefore21)
			offsetInMB = 4 * dataCountInMB
			cksumAfter22, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter22, Equals, cksumBefore22)
			offsetInMB = 5 * dataCountInMB
			cksumAfter23, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter23, Equals, cksumBefore23)

			// Verify chain3
			revertSnapshot(c, spdkCli, snapshotName32, volumeName, engineName, engineFrontendName, replicaAddressMap)
			// Current snapshot tree (with backing image):
			// 	 nil (backing image) -> snap11[0,10] -> snap13[10,30] -> snap15[30,50]
			// 	                                    |\                \
			// 	                                    | \                -> snap23[30,60]
			// 	                                    |  \
			// 	                                    \   -> snap32[10,30] -> head
			// 	                                     \
			// 	                                      -> snap41[10,20] -> snap42[20,30]
			snapshotMap = map[string][]string{
				snapshotName11: {snapshotName13, snapshotName32, snapshotName41},
				snapshotName13: {snapshotName15, snapshotName23},
				snapshotName15: {},
				snapshotName23: {},
				snapshotName32: {types.VolumeHead},
				snapshotName41: {snapshotName42},
				snapshotName42: {},
			}
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName3, replicaName4}, snapshotMap, snapshotOpts, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = 0
			cksumAfter11, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter11, Equals, cksumBefore11)
			offsetInMB = dataCountInMB
			cksumAfter31, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter31, Equals, cksumBefore31)
			offsetInMB = 2 * dataCountInMB
			cksumAfter32, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter32, Equals, cksumBefore32)
			// Verify chain4
			revertSnapshot(c, spdkCli, snapshotName42, volumeName, engineName, engineFrontendName, replicaAddressMap)
			// Current snapshot tree (with backing image):
			// 	 nil (backing image) -> snap11[0,10] -> snap13[10,30] -> snap15[30,50]
			// 	                                    |\                \
			// 	                                    | \                -> snap23[30,60]
			// 	                                    |  \
			// 	                                    \   -> snap32[10,30]
			// 	                                     \
			// 	                                      -> snap41[10,20] -> snap42[20,30] -> head
			snapshotMap = map[string][]string{
				snapshotName11: {snapshotName13, snapshotName32, snapshotName41},
				snapshotName13: {snapshotName15, snapshotName23},
				snapshotName15: {},
				snapshotName23: {},
				snapshotName32: {},
				snapshotName41: {snapshotName42},
				snapshotName42: {types.VolumeHead},
			}
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName3, replicaName4}, snapshotMap, snapshotOpts, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = 0
			cksumAfter11, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter11, Equals, cksumBefore11)
			offsetInMB = dataCountInMB
			cksumAfter41, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter41, Equals, cksumBefore41)
			offsetInMB = 2 * dataCountInMB
			cksumAfter42, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter42, Equals, cksumBefore42)
		}()
	}

	wg.Wait()

	engineFrontendList, err := spdkCli.EngineFrontendList()
	c.Assert(err, IsNil)
	for _, ef := range engineFrontendList {
		err = spdkCli.EngineFrontendDelete(ef.Name)
		c.Assert(err, IsNil)
	}

	engineList, err := spdkCli.EngineList()
	c.Assert(err, IsNil)
	for _, engine := range engineList {
		err = spdkCli.EngineDelete(engine.Name)
		c.Assert(err, IsNil)
	}
	replicaList, err := spdkCli.ReplicaList()
	c.Assert(err, IsNil)
	for _, replica := range replicaList {
		err = spdkCli.ReplicaDelete(replica.Name, true)
		c.Assert(err, IsNil)
	}
}

func (s *TestSuite) TestSPDKMultipleThreadSnapshotOpsAndRebuildingWithoutBackingImage(c *C) {
	fmt.Println("Testing SPDK multiple thread snapshot ops and rebuilding without backing image")
	s.spdkMultipleThreadSnapshotOpsAndRebuilding(c, false)
}

func (s *TestSuite) spdkMultipleThreadFastRebuilding(c *C, withBackingImage bool) {
	diskDriverName := "aio"

	ip, err := commonnet.GetAnyExternalIP()
	c.Assert(err, IsNil)
	err = os.Setenv(commonnet.EnvPodIP, ip)
	c.Assert(err, IsNil)

	ctx, cancel := context.WithCancel(context.Background())
	var spdkWg sync.WaitGroup

	defer func() {
		cancel()
		spdkWg.Wait()
	}()

	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	c.Assert(err, IsNil)
	LaunchTestSPDKGRPCServer(ctx, c, ip, ne.Execute, &spdkWg)

	loopDevicePath := PrepareDiskFile(c)
	defer func() {
		CleanupDiskFile(c, loopDevicePath)
	}()

	spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
	c.Assert(err, IsNil)

	disk, err := spdkCli.DiskCreate(defaultTestDiskName, "", loopDevicePath, diskDriverName, int64(defaultTestBlockSize))
	c.Assert(err, IsNil)
	c.Assert(disk, NotNil)

	disk, err = waitForDiskReady(ctx, spdkCli, loopDevicePath, diskDriverName)
	c.Assert(err, IsNil)
	c.Assert(disk.Path, Equals, loopDevicePath)
	c.Assert(disk.Uuid, Not(Equals), "")

	defer func() {
		err := spdkCli.DiskDelete(defaultTestDiskName, disk.Uuid, disk.Path, diskDriverName)
		c.Assert(err, IsNil)

		disk, err = spdkCli.DiskGet(defaultTestDiskName, disk.Path, diskDriverName)
		c.Assert(err, NotNil)
		c.Assert(disk, IsNil)
	}()

	var bi *api.BackingImage
	if withBackingImage {
		bi, err = spdkCli.BackingImageCreate(defaultTestBackingImageName, defaultTestBackingImageUUID, disk.Uuid, defaultTestBackingImageSize, defaultTestBackingImageChecksum, defaultTestBackingImageDownloadURL, "")
		c.Assert(err, IsNil)
		c.Assert(bi, NotNil)
		defer func() {
			err := spdkCli.BackingImageDelete(defaultTestBackingImageName, disk.Uuid)
			c.Assert(err, IsNil)
		}()

		// check if bi.State is "ready" in 300 seconds
		for i := range maxBackingImageGetRetries {
			bi, err = spdkCli.BackingImageGet(defaultTestBackingImageName, disk.Uuid)
			c.Assert(err, IsNil)

			if bi.State == string(types.BackingImageStateReady) {
				break
			}

			time.Sleep(1 * time.Second)

			if i == maxBackingImageGetRetries-1 {
				c.Assert(bi.State, Equals, string(types.BackingImageStateReady))
			}
		}
	}

	backingImageName := ""
	if withBackingImage {
		backingImageName = bi.Name
	}

	concurrentCount := 5
	dataCountInMB := int64(100)

	// Pre-create all resources concurrently — each goroutine creates the
	// full stack (replicas → engine → engine frontend) for one volume.
	type volumeTestData struct {
		volumeName         string
		engineName         string
		engineFrontendName string
		replicaName1       string
		replicaName2       string
		replica1           *api.Replica
		replicaAddressMap  map[string]string
		endpoint           string
	}

	testVolumes := make([]volumeTestData, concurrentCount)
	createErrs := make([]error, concurrentCount)
	var createWg sync.WaitGroup
	createWg.Add(concurrentCount)
	for i := range concurrentCount {
		go func(idx int) {
			defer createWg.Done()

			vol := &testVolumes[idx]
			vol.volumeName = fmt.Sprintf("test-vol-%d", idx)
			vol.engineName = fmt.Sprintf("%s-e", vol.volumeName)
			vol.engineFrontendName = fmt.Sprintf("%s-ef", vol.volumeName)
			vol.replicaName1 = fmt.Sprintf("%s-replica-1", vol.volumeName)
			vol.replicaName2 = fmt.Sprintf("%s-replica-2", vol.volumeName)

			replica1, err := spdkCli.ReplicaCreate(vol.replicaName1, defaultTestDiskName, disk.Uuid, defaultTestLargeLvolSize, defaultTestReplicaPortCount, backingImageName)
			if err != nil {
				createErrs[idx] = fmt.Errorf("failed to create replica1 for vol %d: %w", idx, err)
				return
			}
			vol.replica1 = replica1

			replica2, err := spdkCli.ReplicaCreate(vol.replicaName2, defaultTestDiskName, disk.Uuid, defaultTestLargeLvolSize, defaultTestReplicaPortCount, backingImageName)
			if err != nil {
				createErrs[idx] = fmt.Errorf("failed to create replica2 for vol %d: %w", idx, err)
				return
			}

			vol.replicaAddressMap = map[string]string{
				replica1.Name: net.JoinHostPort(ip, strconv.Itoa(int(replica1.PortStart))),
				replica2.Name: net.JoinHostPort(ip, strconv.Itoa(int(replica2.PortStart))),
			}

			vol.endpoint = helperutil.GetLonghornDevicePath(vol.volumeName)
			engine, err := spdkCli.EngineCreate(vol.engineName, vol.volumeName, types.FrontendSPDKTCPBlockdev, defaultTestLargeLvolSize, vol.replicaAddressMap, 1, false)
			if err != nil {
				createErrs[idx] = fmt.Errorf("failed to create engine for vol %d: %w", idx, err)
				return
			}

			engineFrontend, err := spdkCli.EngineFrontendCreate(vol.engineFrontendName, vol.volumeName, vol.engineName, types.FrontendSPDKTCPBlockdev, defaultTestLargeLvolSize,
				net.JoinHostPort(engine.IP, strconv.Itoa(int(engine.Port))), 0, 0)
			if err != nil {
				createErrs[idx] = fmt.Errorf("failed to create engine frontend for vol %d: %w", idx, err)
				return
			}
			_ = engineFrontend
		}(i)
	}
	createWg.Wait()
	for i, err := range createErrs {
		c.Assert(err, IsNil, Commentf("concurrent pre-creation failed for vol %d", i))
	}

	wg := sync.WaitGroup{}
	wg.Add(concurrentCount)
	for i := range concurrentCount {
		go func() {
			defer wg.Done()

			vol := testVolumes[i]
			volumeName := vol.volumeName
			engineName := vol.engineName
			engineFrontendName := vol.engineFrontendName
			replicaName1 := vol.replicaName1
			replicaName2 := vol.replicaName2
			replica1 := vol.replica1
			replicaAddressMap := vol.replicaAddressMap
			endpoint := vol.endpoint

			var engine *api.Engine
			var engineFrontend *api.EngineFrontend
			var err error
			// Suppress unused warnings; these are used in the goroutine body below
			_ = engine
			_ = engineFrontend

			// Construct a snapshot tree with enough data before testing rebuilding

			// Build the first chain (chain 1)
			offsetInMB := int64(0)
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore11, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore11, Not(Equals), "")
			snapshotName11 := "snap11"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName11)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName11, false)
			c.Assert(err, IsNil)

			// Current snapshot tree (with backing image):
			//       BackingImage -> snap11 -> head
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore12, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore12, Not(Equals), "")
			snapshotName12 := "snap12"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName12)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName12, false)
			c.Assert(err, IsNil)

			// Current snapshot tree (with backing image):
			//       BackingImage -> snap11 -> snap12 -> head
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12},
					snapshotName12: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = 2 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore13, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore13, Not(Equals), "")
			snapshotName13 := "snap13"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName13)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName13, false)
			c.Assert(err, IsNil)

			// Current snapshot tree (with backing image):
			//       BackingImage -> snap11 -> snap12 -> snap13 -> head
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12},
					snapshotName12: {snapshotName13},
					snapshotName13: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Revert for a new chain (chain 2)
			revertSnapshot(c, spdkCli, snapshotName11, volumeName, engineName, engineFrontendName, replicaAddressMap)

			// Current snapshot tree (with backing image):
			//       BackingImage -> snap11 ->  snap12 -> snap13
			//                          \
			//                           -> head
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12, types.VolumeHead},
					snapshotName12: {snapshotName13},
					snapshotName13: {},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = 1 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore21, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore21, Not(Equals), "")
			snapshotName21 := "snap21"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName21)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName21, false)
			c.Assert(err, IsNil)

			// Current snapshot tree (with backing image):
			//       BackingImage -> snap11 ->  snap12 -> snap13
			//                          \
			//                           -> snap21 -> head
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12, snapshotName21},
					snapshotName12: {snapshotName13},
					snapshotName13: {},
					snapshotName21: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = 2 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBefore22, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore22, Not(Equals), "")
			snapshotName22 := "snap22"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName22)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName22, false)
			c.Assert(err, IsNil)

			// Current snapshot tree (with backing image):
			//       BackingImage -> snap11 ->  snap12 -> snap13
			//                          \
			//                           -> snap21 -> snap22 -> head
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12, snapshotName21},
					snapshotName12: {snapshotName13},
					snapshotName13: {},
					snapshotName21: {snapshotName22},
					snapshotName22: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Revert for a new chain (chain 3)
			revertSnapshot(c, spdkCli, snapshotName21, volumeName, engineName, engineFrontendName, replicaAddressMap)

			// Current snapshot tree (with backing image):
			//       BackingImage -> snap11 ->  snap12 -> snap13
			//                          \
			//                           -> snap21 -> snap22
			//                                 \
			//                                  -> head
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12, snapshotName21},
					snapshotName12: {snapshotName13},
					snapshotName13: {},
					snapshotName21: {snapshotName22, types.VolumeHead},
					snapshotName22: {},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = 2 * dataCountInMB

			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			if err != nil {
				fmt.Printf("Error writing data before creating snap31 for volume %s: %v\n", volumeName, err)
				time.Sleep(60000 * time.Second)
				fmt.Printf("After sleep, still error writing data before creating snap31 for volume %s: %v\n", volumeName, err)
			}
			c.Assert(err, IsNil)
			cksumBefore31, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBefore31, Not(Equals), "")
			snapshotName31 := "snap31"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName31)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName31, false)
			c.Assert(err, IsNil)

			// Current snapshot tree (with backing image):
			//       BackingImage -> snap11 ->  snap12 -> snap13
			//                          \
			//                           -> snap21 -> snap22
			//                                 \
			//                                  -> snap31 -> head
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12, snapshotName21},
					snapshotName12: {snapshotName13},
					snapshotName13: {},
					snapshotName21: {snapshotName22, snapshotName31},
					snapshotName22: {},
					snapshotName31: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Finally, write some data to the head before the rebuilding
			offsetInMB = 3 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			cksumBeforeRebuild11, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumBeforeRebuild11, Not(Equals), "")

			// Current snapshot tree (with backing image):
			//       BackingImage -> snap11[0,1*DATA] -> snap12[1*DATA,2*DATA] -> snap13[2*DATA,3*DATA]
			//                          \
			//                           -> snap21[1*DATA,2*DATA] -> snap22[2*DATA,3*DATA]
			//                                 \
			//                                  -> snap31[2*DATA,3*DATA] -> head[3*DATA,4*DATA]
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12, snapshotName21},
					snapshotName12: {snapshotName13},
					snapshotName13: {},
					snapshotName21: {snapshotName22, snapshotName31},
					snapshotName22: {},
					snapshotName31: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Test online rebuilding

			// Crash replica1
			err = spdkCli.ReplicaDelete(replicaName1, false)
			c.Assert(err, IsNil)
			// TODO: Make replica1 Mode ERR
			//engine, err = spdkCli.EngineGet(engineName)
			//c.Assert(err, IsNil)
			//c.Assert(engine.State, Equals, types.InstanceStateRunning)
			//c.Assert(engine.Frontend, Equals, types.FrontendSPDKTCPBlockdev)
			//c.Assert(engine.Endpoint, Equals, endpoint)
			//c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
			//c.Assert(engine.ReplicaModeMap, DeepEquals, map[string]types.Mode{replicaName1: types.ModeERR, replicaName2: types.ModeRW})

			delete(replicaAddressMap, replicaName1)
			err = spdkCli.EngineReplicaDelete(engineName, replicaName1, net.JoinHostPort(ip, strconv.Itoa(int(replica1.PortStart))))
			c.Assert(err, IsNil)
			engine, err = spdkCli.EngineGet(engineName)
			c.Assert(err, IsNil)
			c.Assert(engine.State, Equals, types.InstanceStateRunning)

			c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
			c.Assert(engine.ReplicaModeMap, DeepEquals, map[string]types.Mode{replicaName2: types.ModeRW})

			replica1, err = spdkCli.ReplicaCreate(replicaName1, defaultTestDiskName, disk.Uuid, defaultTestLargeLvolSize, defaultTestReplicaPortCount, backingImageName)
			c.Assert(err, IsNil)
			c.Assert(replica1.LvsName, Equals, defaultTestDiskName)
			c.Assert(replica1.LvsUUID, Equals, disk.Uuid)
			c.Assert(replica1.State, Equals, types.InstanceStateRunning)
			c.Assert(replica1.PortStart, Not(Equals), int32(0))

			// Before reusing the crashed replica1 for rebuilding, mess up some snapshots and see if the fast rebuilding mechanism can handle it
			replicaTmpAddressMap := map[string]string{
				replica1.Name: net.JoinHostPort(ip, strconv.Itoa(int(replica1.PortStart))),
			}

			err = spdkCli.EngineFrontendDelete(engineFrontendName)
			c.Assert(err, IsNil)
			err = spdkCli.EngineDelete(engineName)
			c.Assert(err, IsNil)
			engine, err = spdkCli.EngineCreate(engineName, volumeName, types.FrontendSPDKTCPBlockdev, defaultTestLargeLvolSize, replicaTmpAddressMap, 1, false)
			c.Assert(err, IsNil)
			c.Assert(engine.State, Equals, types.InstanceStateRunning)
			c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaTmpAddressMap)
			c.Assert(engine.ReplicaModeMap, DeepEquals, map[string]types.Mode{replica1.Name: types.ModeRW})
			c.Assert(engine.Port, Not(Equals), int32(0))

			engineFrontend, err = spdkCli.EngineFrontendCreate(engineFrontendName, volumeName, engineName, types.FrontendSPDKTCPBlockdev, defaultTestLargeLvolSize,
				net.JoinHostPort(engine.IP, strconv.Itoa(int(engine.Port))), 0, 0)
			c.Assert(err, IsNil)
			c.Assert(engineFrontend.Endpoint, Equals, endpoint)

			// Current snapshot tree (with backing image):
			//       BackingImage -> snap11[0,1*DATA] -> snap12[1*DATA,2*DATA] -> snap13[2*DATA,3*DATA]
			//                          \
			//                           -> snap21[1*DATA,2*DATA] -> snap22[2*DATA,3*DATA]
			//                                 \
			//                                  -> snap31[2*DATA,3*DATA] -> head[3*DATA,4*DATA]
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName2},
				map[string][]string{
					snapshotName11: {snapshotName12, snapshotName21},
					snapshotName12: {snapshotName13},
					snapshotName13: {},
					snapshotName21: {snapshotName22, snapshotName31},
					snapshotName22: {},
					snapshotName31: {types.VolumeHead},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Mess up snapshots in chain 1 by corrupting data for snap12 and introducing more invalid snapshots/head for the crashed replica1
			revertSnapshot(c, spdkCli, snapshotName12, volumeName, engineName, engineFrontendName, replicaTmpAddressMap)

			// Current snapshot tree of the crashed replica1 (with backing image):
			//                                               -> head[,]
			//                                              /
			//       BackingImage -> snap11[0,1*DATA] -> snap12[1*DATA,2*DATA] -> snap13[2*DATA,3*DATA]
			//                          \
			//                           -> snap21[1*DATA,2*DATA] -> snap22[2*DATA,3*DATA]
			//                                 \
			//                                  -> snap31[2*DATA,3*DATA] -> head[3*DATA,4*DATA]
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1},
				map[string][]string{
					snapshotName11: {snapshotName12, snapshotName21},
					snapshotName12: {snapshotName13, types.VolumeHead},
					snapshotName13: {},
					snapshotName21: {snapshotName22, snapshotName31},
					snapshotName22: {},
					snapshotName31: {},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			err = spdkCli.EngineFrontendSnapshotDelete(engineFrontendName, snapshotName13)
			c.Assert(err, IsNil)

			// Current snapshot tree of the crashed replica1 (with backing image):
			//                                               -> head[,]
			//                                              /
			//       BackingImage -> snap11[0,1*DATA] -> snap12[1*DATA,2*DATA]
			//                          \
			//                           -> snap21[1*DATA,2*DATA] -> snap22[2*DATA,3*DATA]
			//                                 \
			//                                  -> snap31[2*DATA,3*DATA] -> head[3*DATA,4*DATA]
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1},
				map[string][]string{
					snapshotName11: {snapshotName21, snapshotName12},
					snapshotName12: {types.VolumeHead},
					snapshotName21: {snapshotName22, snapshotName31},
					snapshotName22: {},
					snapshotName31: {},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Try to write invalid data into the snap12 for replica1
			offsetInMB = 2 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			// Since we cannot modify or even delete the snap12 now, we need to put the invalid data to the new snap12-tmp
			snapshotName12Tmp := "snap12-tmp"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName12Tmp)
			c.Assert(err, IsNil)
			// Then deleting snap12 will make ll data be merged into snap12-tmp
			err = spdkCli.EngineFrontendSnapshotDelete(engineFrontendName, snapshotName12)
			c.Assert(err, IsNil)
			// Now we can recreate the snap12 and delete the snap12-tmp
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName12)
			c.Assert(err, IsNil)
			err = spdkCli.EngineFrontendSnapshotDelete(engineFrontendName, snapshotName12Tmp)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName12, false)
			c.Assert(err, IsNil)

			// Current snapshot tree of the crashed replica1 (with backing image):
			//       BackingImage -> snap11[0,1*DATA] -> snap12[1*DATA,2*DATA][2*INVALID_DATA, 3*INVALID_DATA] -> head[,]
			//                          \
			//                           -> snap21[1*DATA,2*DATA] -> snap22[2*DATA,3*DATA]
			//                                 \
			//                                  -> snap31[2*DATA,3*DATA] -> head[3*DATA,4*DATA]
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1},
				map[string][]string{
					snapshotName11: {snapshotName21, snapshotName12},
					snapshotName12: {types.VolumeHead},
					snapshotName21: {snapshotName22, snapshotName31},
					snapshotName22: {},
					snapshotName31: {},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// Then create the invalid snapshot13
			offsetInMB = 2 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName13)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName13, false)
			c.Assert(err, IsNil)

			// Finally, create an extra snapshot14 and leave an invalid head
			offsetInMB = 3 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)
			snapshotName14 := "snap14"
			_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName14)
			c.Assert(err, IsNil)
			err = spdkCli.EngineSnapshotHash(engineName, snapshotName14, false)
			c.Assert(err, IsNil)
			offsetInMB = 4 * dataCountInMB
			err = writeDataToBlockDevice(ne, endpoint, offsetInMB, dataCountInMB)
			c.Assert(err, IsNil)

			// Current snapshot tree of the crashed replica1 (with backing image):
			//       BackingImage -> snap11[0,1*DATA] -> snap12[1*DATA,2*DATA][2*INVALID_DATA, 3*INVALID_DATA] -> snap13[2*INVALID_DATA, 3*INVALID_DATA] -> snap14[3*INVALID_DATA, 4*INVALID_DATA] -> head[4*INVALID_DATA, 5*INVALID_DATA]
			//                          \
			//                           -> snap21[1*DATA,2*DATA] -> snap22[2*DATA,3*DATA]
			//                                 \
			//                                  -> snap31[2*DATA,3*DATA] -> head[3*DATA,4*DATA]
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1},
				map[string][]string{
					snapshotName11: {snapshotName21, snapshotName12},
					snapshotName12: {snapshotName13},
					snapshotName13: {snapshotName14},
					snapshotName14: {types.VolumeHead},
					snapshotName21: {snapshotName22, snapshotName31},
					snapshotName22: {},
					snapshotName31: {},
				},
				nil, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			err = spdkCli.EngineFrontendDelete(engineFrontendName)
			c.Assert(err, IsNil)
			err = spdkCli.EngineDelete(engineName)
			c.Assert(err, IsNil)

			// Relaunch engine with the correct replica
			engine, err = spdkCli.EngineCreate(engineName, volumeName, types.FrontendSPDKTCPBlockdev, defaultTestLargeLvolSize, replicaAddressMap, 1, false)
			c.Assert(err, IsNil)
			c.Assert(engine.State, Equals, types.InstanceStateRunning)
			c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
			c.Assert(engine.ReplicaModeMap, DeepEquals, map[string]types.Mode{replicaName2: types.ModeRW})
			c.Assert(engine.Port, Not(Equals), int32(0))

			engineFrontend, err = spdkCli.EngineFrontendCreate(engineFrontendName, volumeName, engineName, types.FrontendSPDKTCPBlockdev, defaultTestLargeLvolSize,
				net.JoinHostPort(engine.IP, strconv.Itoa(int(engine.Port))), 0, 0)
			c.Assert(err, IsNil)
			c.Assert(engineFrontend.Endpoint, Equals, endpoint)

			// And start the 1st rebuilding and wait for the completion
			err = spdkCli.EngineFrontendReplicaAdd(engineFrontendName, replicaName1, net.JoinHostPort(ip, strconv.Itoa(int(replica1.PortStart))), defaultTestFastSync)
			c.Assert(err, IsNil)
			// The rebuilding should be pretty fast since all existing snapshots and the previous head are there
			WaitForReplicaRebuildingCompleteTimeout(c, spdkCli, engineName, replicaName1, 300)

			// Figure out the 1st rebuilding snapshot name
			replicaAddressMap[replica1.Name] = net.JoinHostPort(ip, strconv.Itoa(int(replica1.PortStart)))
			snapshotNameRebuild1 := ""
			for replicaName := range replicaAddressMap {
				replica, err := spdkCli.ReplicaGet(replicaName)
				c.Assert(err, IsNil)
				for snapName, snapLvol := range replica.Snapshots {
					if strings.HasPrefix(snapName, server.RebuildingSnapshotNamePrefix) {
						c.Assert(snapLvol.Children[types.VolumeHead], Equals, true)
						c.Assert(snapLvol.Parent, Equals, snapshotName31)
						if snapshotNameRebuild1 == "" {
							snapshotNameRebuild1 = snapName
						} else {
							c.Assert(snapName, Equals, snapshotNameRebuild1)
						}
						break
					}
				}
			}
			c.Assert(snapshotNameRebuild1, Not(Equals), "")

			// Verify the 1st rebuilding result
			engine, err = spdkCli.EngineGet(engineName)
			c.Assert(err, IsNil)
			c.Assert(engine.State, Equals, types.InstanceStateRunning)

			c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
			c.Assert(engine.ReplicaModeMap, DeepEquals, map[string]types.Mode{replicaName1: types.ModeRW, replicaName2: types.ModeRW})

			// Rebuilding once leads to snap12 and snap13 reuse (which involve range shallow copy) as well as invalid snap14 deletion
			// Current snapshot tree (with backing image):
			//       BackingImage -> snap11[0,1*DATA] -> snap12[1*DATA,2*DATA] -> snap13[2*DATA,3*DATA]
			//                          \
			//                           -> snap21[1*DATA,2*DATA] -> snap22[2*DATA,3*DATA]
			//                                 \
			//                                  -> snap31[2*DATA,3*DATA] -> rebuild11[3*DATA,4*DATA] -> head[4*DATA,4*DATA]
			snapshotMap := map[string][]string{
				snapshotName11:       {snapshotName12, snapshotName21},
				snapshotName12:       {snapshotName13},
				snapshotName13:       {},
				snapshotName21:       {snapshotName22, snapshotName31},
				snapshotName22:       {},
				snapshotName31:       {snapshotNameRebuild1},
				snapshotNameRebuild1: {types.VolumeHead},
			}
			snapshotOpts := map[string]api.SnapshotOptions{}
			for snapName := range snapshotMap {
				snapshotOpts[snapName] = api.SnapshotOptions{UserCreated: true}
			}
			snapshotOpts[snapshotNameRebuild1] = api.SnapshotOptions{UserCreated: false}

			for snapName := range snapshotMap {
				err = spdkCli.EngineSnapshotHash(engineName, snapName, false)
				c.Assert(err, IsNil)
			}

			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2}, snapshotMap, snapshotOpts, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			// The newly rebuilt replicas should contain correct/unchanged data
			// Verify chain3
			offsetInMB = 0
			cksumAfter11, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter11, Equals, cksumBefore11)
			offsetInMB = dataCountInMB
			cksumAfter21, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter21, Equals, cksumBefore21)
			offsetInMB = 2 * dataCountInMB
			cksumAfter31, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter31, Equals, cksumBefore31)

			// Verify chain2, the volume head will be reverted to the snapshot22
			revertSnapshot(c, spdkCli, snapshotName22, volumeName, engineName, engineFrontendName, replicaAddressMap)
			for snapName := range snapshotMap {
				snapshotOpts[snapName] = api.SnapshotOptions{UserCreated: true}
			}
			snapshotOpts[snapshotNameRebuild1] = api.SnapshotOptions{UserCreated: false}
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11:       {snapshotName12, snapshotName21},
					snapshotName12:       {snapshotName13},
					snapshotName13:       {},
					snapshotName21:       {snapshotName22, snapshotName31},
					snapshotName22:       {types.VolumeHead},
					snapshotName31:       {snapshotNameRebuild1},
					snapshotNameRebuild1: {},
				},
				snapshotOpts, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = 0
			cksumAfter11, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter11, Equals, cksumBefore11)
			offsetInMB = dataCountInMB
			cksumAfter21, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter21, Equals, cksumBefore21)
			offsetInMB = 2 * dataCountInMB
			cksumAfter22, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter22, Equals, cksumBefore22)

			// Verify chain1, the volume head will be reverted to the snapshot13
			revertSnapshot(c, spdkCli, snapshotName13, volumeName, engineName, engineFrontendName, replicaAddressMap)
			for snapName := range snapshotMap {
				snapshotOpts[snapName] = api.SnapshotOptions{UserCreated: true}
			}
			snapshotOpts[snapshotNameRebuild1] = api.SnapshotOptions{UserCreated: false}
			checkReplicaSnapshots(c, spdkCli, engineName, []string{replicaName1, replicaName2},
				map[string][]string{
					snapshotName11:       {snapshotName12, snapshotName21},
					snapshotName12:       {snapshotName13},
					snapshotName13:       {types.VolumeHead},
					snapshotName21:       {snapshotName22, snapshotName31},
					snapshotName22:       {},
					snapshotName31:       {snapshotNameRebuild1},
					snapshotNameRebuild1: {},
				},
				snapshotOpts, checkReplicaSnapshotsMaxRetries, checkReplicaSnapshotsWaitInterval)

			offsetInMB = 0
			cksumAfter11, err = util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter11, Equals, cksumBefore11)
			offsetInMB = dataCountInMB
			cksumAfter12, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter12, Equals, cksumBefore12)
			offsetInMB = 2 * dataCountInMB
			cksumAfter13, err := util.GetFileChunkChecksum(endpoint, offsetInMB*helpertypes.MiB, dataCountInMB*helpertypes.MiB)
			c.Assert(err, IsNil)
			c.Assert(cksumAfter13, Equals, cksumBefore13)
		}()
	}

	wg.Wait()

	engineFrontendList, err := spdkCli.EngineFrontendList()
	c.Assert(err, IsNil)
	for _, ef := range engineFrontendList {
		err = spdkCli.EngineFrontendDelete(ef.Name)
		c.Assert(err, IsNil)
	}

	engineList, err := spdkCli.EngineList()
	c.Assert(err, IsNil)
	for _, engine := range engineList {
		err = spdkCli.EngineDelete(engine.Name)
		c.Assert(err, IsNil)
	}
	replicaList, err := spdkCli.ReplicaList()
	c.Assert(err, IsNil)
	for _, replica := range replicaList {
		err = spdkCli.ReplicaDelete(replica.Name, true)
		c.Assert(err, IsNil)
	}
}

func (s *TestSuite) TestSPDKMultipleThreadFastRebuildingWithoutBackingImage(c *C) {
	fmt.Println("Testing SPDK fast rebuilding with multiple threads without backing image")
	s.spdkMultipleThreadFastRebuilding(c, false)
}

func checkReplicaSnapshots(c *C, spdkCli *client.SPDKClient, engineName string, replicaList []string, snapshotMap map[string][]string, snapshotOpts map[string]api.SnapshotOptions, maxRetries int, retryInterval time.Duration) {
	var lastErr error
	retries := maxRetries
	if retries <= 0 {
		retries = 1
	}
	for attempt := 0; attempt < retries; attempt++ {
		lastErr = nil

		engine, err := spdkCli.EngineGet(engineName)
		if err != nil {
			lastErr = fmt.Errorf("EngineGet failed: %v", err)
		} else {
			for _, replicaName := range replicaList {
				waitReplicaSnapshotChecksum(c, spdkCli, replicaName, "")
			}

			for _, replicaName := range replicaList {
				replica, err := spdkCli.ReplicaGet(replicaName)
				if err != nil {
					lastErr = fmt.Errorf("ReplicaGet failed for %s: %v", replicaName, err)
					break
				}
				if len(replica.Snapshots) != len(snapshotMap) {
					lastErr = fmt.Errorf("replica %s snapshot count mismatch: expected %d, got %d", replicaName, len(snapshotMap), len(replica.Snapshots))
					break
				}

				for snapName, childrenList := range snapshotMap {
					snap := replica.Snapshots[snapName]
					if snap == nil {
						lastErr = fmt.Errorf("snapshot %s not found in replica %s", snapName, replicaName)
						break
					}
					engineSnap := engine.Snapshots[snapName]
					if engineSnap == nil {
						lastErr = fmt.Errorf("snapshot %s not found in engine", snapName)
						break
					}
					if !reflect.DeepEqual(engineSnap.Children, snap.Children) {
						lastErr = fmt.Errorf("snapshot %s children mismatch", snapName)
						break
					}
					for _, childSnapName := range childrenList {
						ok, exists := snap.Children[childSnapName]
						if !exists || !ok {
							lastErr = fmt.Errorf("child snapshot %s not present in snapshot %s", childSnapName, snapName)
							break
						}
						if childSnapName != types.VolumeHead {
							childSnap := replica.Snapshots[childSnapName]
							if childSnap == nil {
								lastErr = fmt.Errorf("child snapshot %s not found in replica %s", childSnapName, replicaName)
								break
							}
							if childSnap.Parent != snapName {
								lastErr = fmt.Errorf("child snapshot %s parent mismatch: expected %s, got %s", childSnapName, snapName, childSnap.Parent)
								break
							}
						}
					}
					if lastErr != nil {
						break
					}
				}
				if lastErr != nil {
					break
				}
				for snapName, opts := range snapshotOpts {
					snap := replica.Snapshots[snapName]
					if snap == nil {
						lastErr = fmt.Errorf("snapshot %s not found in replica %s", snapName, replicaName)
						break
					}
					engineSnap := engine.Snapshots[snapName]
					if engineSnap.UserCreated != opts.UserCreated {
						lastErr = fmt.Errorf("snapshot %s UserCreated mismatch: expected %v, got %v", snapName, opts.UserCreated, engineSnap.UserCreated)
						break
					}
				}
				if lastErr != nil {
					break
				}
			}
		}

		if lastErr == nil {
			return
		}

		if attempt < retries-1 {
			time.Sleep(retryInterval)
		}
	}

	c.Assert(lastErr, IsNil)
}

func waitReplicaSnapshotChecksum(c *C, spdkCli *client.SPDKClient, replicaName, targetSnapName string) {
	waitReplicaSnapshotChecksumTimeout(c, spdkCli, replicaName, targetSnapName, defaultTestSnapChecksumWaitCount)
}

func waitReplicaSnapshotChecksumTimeout(c *C, spdkCli *client.SPDKClient, replicaName, targetSnapName string, timeoutInSecond int) {
	ticker := time.NewTicker(defaultTestSnapChecksumWaitInterval)
	defer ticker.Stop()
	timer := time.NewTimer(time.Duration(timeoutInSecond) * time.Second)
	defer timer.Stop()

	hasChecksum := true
	for {
		select {
		case <-timer.C:
			c.Assert(hasChecksum, Equals, true)
			return
		case <-ticker.C:
			hasChecksum = true
			replica, err := spdkCli.ReplicaGet(replicaName)
			c.Assert(err, IsNil)
			if targetSnapName == "" || replica.Snapshots[targetSnapName] != nil {
				for snapName, snap := range replica.Snapshots {
					if targetSnapName == "" || snapName == targetSnapName {
						if !snap.UserCreated {
							continue
						}
						if snap.SnapshotChecksum == "" {
							hasChecksum = false
							break
						}
					}
				}
			}
		}
		if hasChecksum {
			break
		}
	}

	c.Assert(hasChecksum, Equals, true)
}

func revertSnapshot(c *C, spdkCli *client.SPDKClient, snapshotName, volumeName, engineName, engineFrontendName string, replicaAddressMap map[string]string) {
	ip, err := commonnet.GetAnyExternalIP()
	c.Assert(err, IsNil)
	err = os.Setenv(commonnet.EnvPodIP, ip)
	c.Assert(err, IsNil)

	engine, err := spdkCli.EngineGet(engineName)
	c.Assert(err, IsNil)

	if engine.State != types.InstanceStateRunning {
		return
	}
	volumeSize := engine.SpecSize

	engineFrontends, err := spdkCli.EngineFrontendList()
	c.Assert(err, IsNil)
	var prevFrontend, prevEndpoint string
	if ef, ok := engineFrontends[engineFrontendName]; ok {
		prevFrontend = ef.Frontend
		prevEndpoint = ef.Endpoint
	} else {
		prevFrontend = types.FrontendEmpty
	}

	if prevFrontend != types.FrontendEmpty {
		// Restart the engine without the frontend
		err = spdkCli.EngineFrontendDelete(engineFrontendName)
		c.Assert(err, IsNil)
		err = spdkCli.EngineDelete(engineName)
		c.Assert(err, IsNil)
		engine, err = spdkCli.EngineCreate(engineName, volumeName, types.FrontendEmpty, volumeSize, replicaAddressMap, 1, false)
		c.Assert(err, IsNil)
		c.Assert(engine.State, Equals, types.InstanceStateRunning)
		c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
		c.Assert(engine.Port, Equals, int32(0))
	}

	err = spdkCli.EngineSnapshotRevert(engineName, snapshotName)
	c.Assert(err, IsNil)

	if prevFrontend != types.FrontendEmpty {
		// Restart the engine with the previous frontend
		err = spdkCli.EngineFrontendDelete(engineFrontendName)
		c.Assert(err, IsNil)
		err = spdkCli.EngineDelete(engineName)
		c.Assert(err, IsNil)
		engine, err = spdkCli.EngineCreate(engineName, volumeName, prevFrontend, volumeSize, replicaAddressMap, 1, false)
		c.Assert(err, IsNil)
		c.Assert(engine.State, Equals, types.InstanceStateRunning)
		c.Assert(engine.ReplicaAddressMap, DeepEquals, replicaAddressMap)
		c.Assert(engine.Port, Not(Equals), int32(0))

		engineFrontend, err := spdkCli.EngineFrontendCreate(engineFrontendName, volumeName, engineName, prevFrontend, volumeSize,
			net.JoinHostPort(engine.IP, strconv.Itoa(int(engine.Port))), 0, 0)
		c.Assert(err, IsNil)
		c.Assert(engineFrontend.Endpoint, Equals, prevEndpoint)
	}
}

func WaitForReplicaRebuildingComplete(c *C, spdkCli *client.SPDKClient, engineName, replicaName string) {
	WaitForReplicaRebuildingCompleteTimeout(c, spdkCli, engineName, replicaName, defaultTestRebuildingWaitCount)
}

func WaitForReplicaRebuildingCompleteTimeout(c *C, spdkCli *client.SPDKClient, engineName, replicaName string, timeoutInSecond int) {
	complete := false

	for cnt := 0; cnt < timeoutInSecond; cnt++ {
		rebuildingStatus, err := spdkCli.ReplicaRebuildingDstShallowCopyCheck(replicaName)
		c.Assert(err, IsNil)
		c.Assert(rebuildingStatus.Error, Equals, "")
		switch rebuildingStatus.State {
		case "":
			c.Assert(rebuildingStatus.SnapshotName, Equals, "")
			c.Assert(rebuildingStatus.TotalState, Equals, "")
			c.Assert(rebuildingStatus.Progress, Equals, uint32(0))
			c.Assert(rebuildingStatus.TotalProgress, Equals, uint32(0))
			c.Assert(rebuildingStatus.TotalState, Equals, "")
		case types.ProgressStateStarting:
			c.Assert(rebuildingStatus.SnapshotName, Equals, "")
			c.Assert(rebuildingStatus.TotalState, Equals, "")
			c.Assert(rebuildingStatus.Progress, Equals, uint32(0))
			c.Assert(rebuildingStatus.TotalProgress, Equals, uint32(0))
			c.Assert(rebuildingStatus.TotalState, Equals, types.ProgressStateInProgress)
		case types.ProgressStateInProgress:
			c.Assert(rebuildingStatus.SnapshotName, Not(Equals), "")
			c.Assert(rebuildingStatus.TotalState, Equals, types.ProgressStateInProgress)
			c.Assert(rebuildingStatus.Progress <= 100, Equals, true)
			c.Assert(rebuildingStatus.TotalProgress < 100, Equals, true)
		case types.ProgressStateComplete:
			c.Assert(rebuildingStatus.Progress, Equals, uint32(100))
			if rebuildingStatus.TotalState == types.ProgressStateInProgress {
				c.Assert(rebuildingStatus.TotalProgress <= 100, Equals, true)
			} else {
				c.Assert(rebuildingStatus.TotalState, Equals, types.ProgressStateComplete)
				c.Assert(rebuildingStatus.TotalProgress, Equals, uint32(100))
			}
		default:
			c.Fatalf("Unexpected rebuilding state %v", rebuildingStatus.State)
		}

		if rebuildingStatus.TotalState == types.ProgressStateComplete {
			engine, err := spdkCli.EngineGet(engineName)
			c.Assert(err, IsNil)
			c.Assert(engine.State, Equals, types.InstanceStateRunning)
			if engine.ReplicaModeMap[replicaName] == types.ModeRW {
				complete = true
				break
			}
		}

		time.Sleep(defaultTestRebuildingWaitInterval)
	}

	c.Assert(complete, Equals, true)
}

func (s *TestSuite) TestSPDKEngineFrontendReplicaAdd(c *C) {
	fmt.Println("Testing SPDK engine frontend replica add with data verification")

	diskDriverName := "aio"

	ip, err := commonnet.GetAnyExternalIP()
	c.Assert(err, IsNil)
	err = os.Setenv(commonnet.EnvPodIP, ip)
	c.Assert(err, IsNil)

	ctx, cancel := context.WithCancel(context.Background())
	var spdkWg sync.WaitGroup
	defer func() {
		cancel()
		spdkWg.Wait()
	}()

	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	c.Assert(err, IsNil)
	LaunchTestSPDKGRPCServer(ctx, c, ip, ne.Execute, &spdkWg)

	loopDevicePath := PrepareDiskFile(c)
	defer func() {
		CleanupDiskFile(c, loopDevicePath)
	}()

	spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
	c.Assert(err, IsNil)
	defer func() {
		if errClose := spdkCli.Close(); errClose != nil {
			logrus.WithError(errClose).Error("Failed to close SPDK client")
		}
	}()

	disk, err := spdkCli.DiskCreate(defaultTestDiskName, "", loopDevicePath, diskDriverName, int64(defaultTestBlockSize))
	c.Assert(err, IsNil)
	c.Assert(disk, NotNil)

	disk, err = waitForDiskReady(ctx, spdkCli, loopDevicePath, diskDriverName)
	c.Assert(err, IsNil)
	c.Assert(disk.Path, Equals, loopDevicePath)
	c.Assert(disk.Uuid, Not(Equals), "")

	defer func() {
		err := spdkCli.DiskDelete(defaultTestDiskName, disk.Uuid, disk.Path, diskDriverName)
		c.Assert(err, IsNil)
	}()

	volumeName := getVolumeName()
	engineName := fmt.Sprintf("%s-e", volumeName)
	engineFrontendName := fmt.Sprintf("%s-ef", volumeName)
	replicaNames := []string{
		fmt.Sprintf("%s-replica-1", volumeName),
		fmt.Sprintf("%s-replica-2", volumeName),
	}
	replicas := make(map[string]*api.Replica)

	defer func() {
		err = spdkCli.EngineFrontendDelete(engineFrontendName)
		c.Assert(err, IsNil)

		err = spdkCli.EngineDelete(engineName)
		c.Assert(err, IsNil)

		for _, replica := range replicas {
			err = spdkCli.ReplicaDelete(replica.Name, true)
			c.Assert(err, IsNil)
		}
	}()

	// 1. Create first replica
	replica, err := spdkCli.ReplicaCreate(replicaNames[0], defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
	c.Assert(err, IsNil)
	replicas[replicaNames[0]] = replica

	replicaAddressMap := make(map[string]string)
	replicaAddressMap[replicaNames[0]] = net.JoinHostPort(ip, strconv.Itoa(int(replica.PortStart)))

	// 2. Create Engine with 1 replica
	engine, err := spdkCli.EngineCreate(engineName, volumeName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize, replicaAddressMap, 1, false)
	c.Assert(err, IsNil)

	// 3. Create Engine Frontend
	engineFrontend, err := spdkCli.EngineFrontendCreate(engineFrontendName, volumeName, engineName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize,
		net.JoinHostPort(engine.IP, strconv.Itoa(int(engine.Port))), 0, 0)
	c.Assert(err, IsNil)
	c.Assert(engineFrontend.State, Equals, types.InstanceStateRunning)

	endpoint := helperutil.GetLonghornDevicePath(volumeName)
	c.Assert(engineFrontend.Endpoint, Equals, endpoint)

	// 4. Write Data to Volume (Pattern A)
	dataA := writePatternToBlockDevice(c, endpoint, 'A', 0, 4096)

	// 5. Take Snapshot
	snapshotName := "snap1"
	_, err = spdkCli.EngineFrontendSnapshotCreate(engineFrontendName, snapshotName)
	c.Assert(err, IsNil)

	// 6. Write Data to Volume (Pattern B)
	dataB := writePatternToBlockDevice(c, endpoint, 'B', 4096, 4096)

	// 7. Add Second Replica
	replica2, err := spdkCli.ReplicaCreate(replicaNames[1], defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
	c.Assert(err, IsNil)
	replicas[replicaNames[1]] = replica2
	replica2Address := net.JoinHostPort(ip, strconv.Itoa(int(replica2.PortStart)))

	err = spdkCli.EngineFrontendReplicaAdd(engineFrontendName, replicaNames[1], replica2Address, defaultTestFastSync)
	c.Assert(err, IsNil)

	// 8. Wait for Replica Add to Complete
	err = retry.Do(func() error {
		e, err := spdkCli.EngineGet(engineName)
		if err != nil {
			return err
		}
		if e.ReplicaModeMap[replicaNames[1]] != types.ModeRW {
			return fmt.Errorf("replica %s is not RW yest: %v", replicaNames[1], e.ReplicaModeMap[replicaNames[1]])
		}
		return nil
	}, retry.Delay(1*time.Second), retry.Attempts(60))
	c.Assert(err, IsNil)

	// 9. Verify rebuilt replica works with the engine by removing replica 1
	// Delete replica 1 from the engine so the engine runs solely on the rebuilt replica 2,
	// then read data through the frontend endpoint to verify correctness end-to-end.
	replica1Address := replicaAddressMap[replicaNames[0]]
	err = spdkCli.EngineReplicaDelete(engineName, replicaNames[0], replica1Address)
	c.Assert(err, IsNil)

	err = spdkCli.ReplicaDelete(replicaNames[0], true)
	c.Assert(err, IsNil)
	delete(replicas, replicaNames[0])

	// Read back and verify Data A and B through the engine frontend
	readAndVerifyBlockDevicePattern(c, endpoint, dataA, 0)
	readAndVerifyBlockDevicePattern(c, endpoint, dataB, 4096)
}

// TestSPDKEngineFrontendReplicaAddErrorHandling tests the error handling of replica addition.
// It verifies that errors during shallow copy and finish phases are correctly reported and handled,
// and that the operation can be retried successfully.
func (s *TestSuite) TestSPDKEngineFrontendReplicaAddErrorHandling(c *C) {

	fmt.Println("Testing SPDK engine frontend replica add error handling")

	diskDriverName := "aio"

	ip, err := commonnet.GetAnyExternalIP()
	c.Assert(err, IsNil)
	err = os.Setenv(commonnet.EnvPodIP, ip)
	c.Assert(err, IsNil)

	ctx, cancel := context.WithCancel(context.Background())
	var spdkWg sync.WaitGroup
	defer func() {
		cancel()
		spdkWg.Wait()
	}()

	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	c.Assert(err, IsNil)
	// Use launchTestSPDKGRPCServer helper to get access to the server instance
	srv := launchTestSPDKGRPCServer(ctx, c, ip, ne.Execute, &spdkWg)

	loopDevicePath := PrepareDiskFile(c)
	defer func() {
		CleanupDiskFile(c, loopDevicePath)
	}()

	spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
	c.Assert(err, IsNil)
	defer func() {
		if errClose := spdkCli.Close(); errClose != nil {
			logrus.WithError(errClose).Error("Failed to close SPDK client")
		}
	}()

	disk, err := spdkCli.DiskCreate(defaultTestDiskName, "", loopDevicePath, diskDriverName, int64(defaultTestBlockSize))
	c.Assert(err, IsNil)
	c.Assert(disk, NotNil)

	disk, err = waitForDiskReady(ctx, spdkCli, loopDevicePath, diskDriverName)
	c.Assert(err, IsNil)
	c.Assert(disk.Path, Equals, loopDevicePath)
	c.Assert(disk.Uuid, Not(Equals), "")

	defer func() {
		err := spdkCli.DiskDelete(defaultTestDiskName, disk.Uuid, disk.Path, diskDriverName)
		c.Assert(err, IsNil)
	}()

	volumeName := fmt.Sprintf("test-err-vol-%s", time.Now().Format("20060102150405"))
	engineName := fmt.Sprintf("%s-e", volumeName)
	engineFrontendName := fmt.Sprintf("%s-ef", volumeName)
	replicaNames := []string{
		fmt.Sprintf("%s-replica-1", volumeName),
		fmt.Sprintf("%s-replica-2", volumeName),
	}
	replicas := make(map[string]*api.Replica)

	defer func() {
		// Cleanup: Try to delete frontend if exists
		_ = spdkCli.EngineFrontendDelete(engineFrontendName)

		err = spdkCli.EngineDelete(engineName)
		c.Assert(err, IsNil)

		for _, replica := range replicas {
			err = spdkCli.ReplicaDelete(replica.Name, true)
			c.Assert(err, IsNil)
		}
	}()

	// 1. Create first replica
	replica, err := spdkCli.ReplicaCreate(replicaNames[0], defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
	c.Assert(err, IsNil)
	replicas[replicaNames[0]] = replica

	replicaAddressMap := make(map[string]string)
	replicaAddressMap[replicaNames[0]] = net.JoinHostPort(ip, strconv.Itoa(int(replica.PortStart)))

	// 2. Create Engine with 1 replica
	engine, err := spdkCli.EngineCreate(engineName, volumeName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize, replicaAddressMap, 1, false)
	c.Assert(err, IsNil)

	// 3. Create Engine Frontend
	engineFrontend, err := spdkCli.EngineFrontendCreate(engineFrontendName, volumeName, engineName, types.FrontendSPDKTCPBlockdev, defaultTestLvolSize,
		net.JoinHostPort(engine.IP, strconv.Itoa(int(engine.Port))), 0, 0)
	c.Assert(err, IsNil)
	c.Assert(engineFrontend.State, Equals, types.InstanceStateRunning)

	endpoint := helperutil.GetLonghornDevicePath(volumeName)

	// Get Internal Engine Struct to inject errors
	internalEngine := srv.GetEngineStruct(engineName)
	c.Assert(internalEngine, NotNil)

	// 4. Create Second Replica
	replica2, err := spdkCli.ReplicaCreate(replicaNames[1], defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
	c.Assert(err, IsNil)
	replicas[replicaNames[1]] = replica2
	replica2Address := net.JoinHostPort(ip, strconv.Itoa(int(replica2.PortStart)))

	// Helper to wait for engine replica to reach ERR mode
	// (indicates the async replica add goroutine on the Engine side has failed).
	// The Engine sets the mode to ERR before running SPDK cleanup, so this
	// returns quickly without waiting for detach timeouts.
	waitForReplicaERR := func(replicaName string) {
		err = retry.Do(func() error {
			e, err := spdkCli.EngineGet(engineName)
			if err != nil {
				return err
			}
			mode, ok := e.ReplicaModeMap[replicaName]
			if !ok {
				return fmt.Errorf("replica %s not found in engine mode map", replicaName)
			}
			if mode != types.ModeERR {
				return fmt.Errorf("replica %s mode is %v, expected ERR", replicaName, mode)
			}
			return nil
		}, retry.Delay(500*time.Millisecond), retry.Attempts(30))
		c.Assert(err, IsNil)
	}

	// 5. Test Shallow Copy Error
	internalEngine.SetReplicaAdder(&server.MockReplicaAdder{
		ShallowCopyFunc: func(srcReplicaServiceCli *client.SPDKClient, srcReplicaName, dstReplicaName string, snapshots []*api.Lvol, fastSync bool) error {
			return fmt.Errorf("injected shallow copy error")
		},
	})

	// Call ReplicaAdd (async)
	err = spdkCli.EngineFrontendReplicaAdd(engineFrontendName, replicaNames[1], replica2Address, defaultTestFastSync)
	c.Assert(err, IsNil) // Should be nil as it returns immediately

	waitForReplicaERR(replicaNames[1])

	// Reset adder
	internalEngine.SetReplicaAdder(nil)

	err = spdkCli.EngineReplicaDelete(engineName, replicaNames[1], replica2Address)
	c.Assert(err, IsNil)

	// The engine's ReplicaAdd goroutine internally calls replicaAddFinish on shallow copy
	// failure, which detaches the external snapshot NVMe controller and stops the source from
	// exposing. So we only need to delete/recreate the dst replica for a clean state.
	err = spdkCli.ReplicaDelete(replicaNames[1], true)
	c.Assert(err, IsNil)

	replica2, err = spdkCli.ReplicaCreate(replicaNames[1], defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
	c.Assert(err, IsNil)
	replica2Address = net.JoinHostPort(ip, strconv.Itoa(int(replica2.PortStart)))

	// 5b. Test Shallow Copy Error with Replica in Error State
	// This tests the production scenario where a real shallow copy failure sets the
	// replica's internal state to Error (via RebuildingDstShallowCopyStart's defer).
	// The test hook operates at Engine level and doesn't trigger per-replica error state,
	// so we use SetTestErrorState to simulate the production behavior.
	// Without the P0 fix, RebuildingDstFinish would reject error-state replicas,
	// causing doCleanupForRebuildingDst to never run, leaving the external snapshot
	// NVMe controller attached and causing bdev_nvme_detach_controller to hang.
	internalEngine.SetReplicaAdder(&server.MockReplicaAdder{
		ShallowCopyFunc: func(dstReplicaServiceCli *client.SPDKClient, srcReplicaName, dstReplicaName string, snapshots []*api.Lvol, fastSync bool) error {
			// Simulate what happens in production: the per-replica error state is set
			// by RebuildingDstShallowCopyStart's defer when the actual shallow copy fails.
			internalReplica := srv.GetReplicaStruct(dstReplicaName)
			c.Assert(internalReplica, NotNil)
			internalReplica.SetTestErrorState("simulated production shallow copy failure")
			return fmt.Errorf("injected shallow copy error with replica error state")
		},
	})

	err = spdkCli.EngineFrontendReplicaAdd(engineFrontendName, replicaNames[1], replica2Address, defaultTestFastSync)
	c.Assert(err, IsNil)

	waitForReplicaERR(replicaNames[1])

	// Reset adder
	internalEngine.SetReplicaAdder(nil)

	// Clean up the partial state in Engine
	err = spdkCli.EngineReplicaDelete(engineName, replicaNames[1], replica2Address)
	c.Assert(err, IsNil)

	// ReplicaDelete should succeed without hanging, because even though the replica
	// was in error state, RebuildingDstFinish (with the P0 fix) still performed
	// doCleanupForRebuildingDst, disconnecting the external snapshot NVMe controller.
	err = spdkCli.ReplicaDelete(replicaNames[1], true)
	c.Assert(err, IsNil)

	replica2, err = spdkCli.ReplicaCreate(replicaNames[1], defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
	c.Assert(err, IsNil)
	replica2Address = net.JoinHostPort(ip, strconv.Itoa(int(replica2.PortStart)))

	// 6. Test Finish Error
	// The mock FinishFunc is responsible for calling Real.ReplicaAddFinish()
	// to clean up SPDK resources before returning the injected error.
	// The engine goroutine simply marks the replica as ERR without fallback cleanup.
	finishErrMock := &server.MockReplicaAdder{}
	finishErrMock.FinishFunc = func(srcReplicaServiceCli *client.SPDKClient, dstReplicaServiceCli *client.SPDKClient, srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress string) error {
		// Clean up SPDK resources via the real finish before returning error
		_ = finishErrMock.Real.ReplicaAddFinish(srcReplicaServiceCli, dstReplicaServiceCli, srcReplicaName, srcReplicaAddress, dstReplicaName, dstReplicaAddress)
		return fmt.Errorf("injected finish error")
	}
	internalEngine.SetReplicaAdder(finishErrMock)

	err = spdkCli.EngineFrontendReplicaAdd(engineFrontendName, replicaNames[1], replica2Address, defaultTestFastSync)
	c.Assert(err, IsNil)

	waitForReplicaERR(replicaNames[1])

	// Reset adder
	internalEngine.SetReplicaAdder(nil)

	// Clean up the partial state in Engine
	err = spdkCli.EngineReplicaDelete(engineName, replicaNames[1], replica2Address)
	c.Assert(err, IsNil)

	// The mock's FinishFunc called Real.ReplicaAddFinish() for SPDK cleanup,
	// so resources are already cleaned up and ReplicaDelete won't hang.
	err = spdkCli.ReplicaDelete(replicaNames[1], true)
	c.Assert(err, IsNil)
	replica2, err = spdkCli.ReplicaCreate(replicaNames[1], defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
	c.Assert(err, IsNil)
	replica2Address = net.JoinHostPort(ip, strconv.Itoa(int(replica2.PortStart)))

	// 6b. Test Engine Lock is Released During replicaAddFinish Phase 2 (RPC calls)
	// This verifies the 3-phase lock refactoring: the Engine lock should NOT be held
	// during Phase 2 when RPC calls (ReplicaRebuildingSrcFinish, ReplicaRebuildingDstFinish)
	// are executed. Without this refactoring, these RPCs would block all Engine operations
	// for 10+ seconds on ETIMEDOUT.
	phase2LockReleased := make(chan bool, 1)
	internalEngine.SetReplicaAddFinishUnlockedHook(func() {
		// This hook runs inside Phase 2 of replicaAddFinish, where the Engine lock
		// should be released. Verify by trying to acquire the lock.
		if internalEngine.TryLock() {
			// Lock was free — 3-phase pattern is working correctly
			internalEngine.Unlock()
			phase2LockReleased <- true
		} else {
			// Lock was held — 3-phase pattern is NOT working (old behavior)
			phase2LockReleased <- false
		}
	})

	err = spdkCli.EngineFrontendReplicaAdd(engineFrontendName, replicaNames[1], replica2Address, defaultTestFastSync)
	c.Assert(err, IsNil)

	// Wait for the Phase 2 hook to fire and report lock status
	select {
	case lockReleased := <-phase2LockReleased:
		c.Assert(lockReleased, Equals, true)
	case <-time.After(60 * time.Second):
		c.Fatal("Timed out waiting for replicaAddFinish Phase 2 hook to fire")
	}

	// Reset hook
	internalEngine.SetReplicaAddFinishUnlockedHook(nil)

	// Wait for Replica Add to Complete
	err = retry.Do(func() error {
		e, err := spdkCli.EngineGet(engineName)
		if err != nil {
			return err
		}
		if e.ReplicaModeMap[replicaNames[1]] != types.ModeRW {
			return fmt.Errorf("replica %s is not RW yet: %v", replicaNames[1], e.ReplicaModeMap[replicaNames[1]])
		}
		return nil
	}, retry.Delay(1*time.Second), retry.Attempts(60))
	c.Assert(err, IsNil)

	// 6c. Test MockReplicaAdder fallback always uses the real adder.
	// Install a failing mock first, then replace it with a mock that leaves all
	// hooks nil. The nil-hook mock must fall through to the real implementation,
	// not the previously installed mock.
	err = spdkCli.EngineReplicaDelete(engineName, replicaNames[1], replica2Address)
	c.Assert(err, IsNil)
	err = spdkCli.ReplicaDelete(replicaNames[1], true)
	c.Assert(err, IsNil)

	replica2, err = spdkCli.ReplicaCreate(replicaNames[1], defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
	c.Assert(err, IsNil)
	replica2Address = net.JoinHostPort(ip, strconv.Itoa(int(replica2.PortStart)))
	replicas[replicaNames[1]] = replica2

	internalEngine.SetReplicaAdder(&server.MockReplicaAdder{
		ShallowCopyFunc: func(dstReplicaServiceCli *client.SPDKClient, srcReplicaName, dstReplicaName string, snapshots []*api.Lvol, fastSync bool) error {
			return fmt.Errorf("stale mock should not be used as fallback")
		},
	})
	internalEngine.SetReplicaAdder(&server.MockReplicaAdder{})

	err = spdkCli.EngineFrontendReplicaAdd(engineFrontendName, replicaNames[1], replica2Address, defaultTestFastSync)
	c.Assert(err, IsNil)

	err = retry.Do(func() error {
		e, err := spdkCli.EngineGet(engineName)
		if err != nil {
			return err
		}
		if e.ReplicaModeMap[replicaNames[1]] != types.ModeRW {
			return fmt.Errorf("replica %s is not RW yet after fallback test: %v", replicaNames[1], e.ReplicaModeMap[replicaNames[1]])
		}
		return nil
	}, retry.Delay(1*time.Second), retry.Attempts(60))
	c.Assert(err, IsNil)

	internalEngine.SetReplicaAdder(nil)

	// Verify Data I/O is still working
	f, err := os.OpenFile(endpoint, os.O_RDWR, 0666)
	c.Assert(err, IsNil)
	data := []byte("phase2-lock-test")
	_, err = f.WriteAt(data, 0)
	c.Assert(err, IsNil)
	err = f.Close()
	c.Assert(err, IsNil)
}

func (s *TestSuite) TestSPDKEngineReplicaAddWithoutEngineFrontendInfo(c *C) {
	diskDriverName := "aio"

	ip, err := commonnet.GetAnyExternalIP()
	c.Assert(err, IsNil)
	err = os.Setenv(commonnet.EnvPodIP, ip)
	c.Assert(err, IsNil)

	ctx, cancel := context.WithCancel(context.Background())
	var spdkWg sync.WaitGroup
	defer func() {
		cancel()
		spdkWg.Wait()
	}()

	ne, err := helperutil.NewExecutor(commontypes.ProcDirectory)
	c.Assert(err, IsNil)
	LaunchTestSPDKGRPCServer(ctx, c, ip, ne.Execute, &spdkWg)

	loopDevicePath := PrepareDiskFile(c)
	defer func() {
		CleanupDiskFile(c, loopDevicePath)
	}()

	spdkCli, err := client.NewSPDKClient(net.JoinHostPort(ip, strconv.Itoa(types.SPDKServicePort)))
	c.Assert(err, IsNil)
	defer func() {
		if errClose := spdkCli.Close(); errClose != nil {
			logrus.WithError(errClose).Error("Failed to close SPDK client")
		}
	}()

	disk, err := spdkCli.DiskCreate(defaultTestDiskName, "", loopDevicePath, diskDriverName, int64(defaultTestBlockSize))
	c.Assert(err, IsNil)
	c.Assert(disk, NotNil)

	disk, err = waitForDiskReady(ctx, spdkCli, loopDevicePath, diskDriverName)
	c.Assert(err, IsNil)

	defer func() {
		err := spdkCli.DiskDelete(defaultTestDiskName, disk.Uuid, disk.Path, diskDriverName)
		c.Assert(err, IsNil)
	}()

	volumeName := fmt.Sprintf("test-direct-replica-add-%s", time.Now().Format("20060102150405"))
	engineName := fmt.Sprintf("%s-e", volumeName)
	replicaNames := []string{
		fmt.Sprintf("%s-replica-1", volumeName),
		fmt.Sprintf("%s-replica-2", volumeName),
	}
	replicas := make(map[string]*api.Replica)

	defer func() {
		_ = spdkCli.EngineDelete(engineName)
		for _, replica := range replicas {
			_ = spdkCli.ReplicaDelete(replica.Name, true)
		}
	}()

	replica1, err := spdkCli.ReplicaCreate(replicaNames[0], defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
	c.Assert(err, IsNil)
	replicas[replicaNames[0]] = replica1
	replicaAddressMap := map[string]string{
		replicaNames[0]: net.JoinHostPort(ip, strconv.Itoa(int(replica1.PortStart))),
	}

	_, err = spdkCli.EngineCreate(engineName, volumeName, types.FrontendEmpty, defaultTestLvolSize, replicaAddressMap, 1, false)
	c.Assert(err, IsNil)

	replica2, err := spdkCli.ReplicaCreate(replicaNames[1], defaultTestDiskName, disk.Uuid, defaultTestLvolSize, defaultTestReplicaPortCount, "")
	c.Assert(err, IsNil)
	replicas[replicaNames[1]] = replica2
	replica2Address := net.JoinHostPort(ip, strconv.Itoa(int(replica2.PortStart)))

	err = spdkCli.EngineReplicaAdd(engineName, replicaNames[1], replica2Address, defaultTestFastSync, "", "")
	c.Assert(err, IsNil)

	err = retry.Do(func() error {
		e, err := spdkCli.EngineGet(engineName)
		if err != nil {
			return err
		}
		if e.ReplicaModeMap[replicaNames[1]] != types.ModeRW {
			return fmt.Errorf("replica %s is not RW yet: %v", replicaNames[1], e.ReplicaModeMap[replicaNames[1]])
		}
		return nil
	}, retry.Delay(1*time.Second), retry.Attempts(60))
	c.Assert(err, IsNil)
}

// writePatternToBlockDevice writes a repeated byte pattern to a block device at the given
// byte offset and returns the written data slice for later verification.
func writePatternToBlockDevice(c *C, endpoint string, pattern byte, offset, length int64) []byte {
	data := make([]byte, length)
	for i := range data {
		data[i] = pattern
	}
	f, err := os.OpenFile(endpoint, os.O_RDWR, 0666)
	c.Assert(err, IsNil)
	_, err = f.WriteAt(data, offset)
	c.Assert(err, IsNil)
	err = f.Sync()
	c.Assert(err, IsNil)
	err = f.Close()
	c.Assert(err, IsNil)
	return data
}

// readAndVerifyBlockDevicePattern reads data from a block device and verifies it matches
// the expected data using DeepEquals.
func readAndVerifyBlockDevicePattern(c *C, endpoint string, expected []byte, offset int64) {
	f, err := os.OpenFile(endpoint, os.O_RDONLY, 0666)
	c.Assert(err, IsNil)
	defer func() {
		err = f.Close()
		c.Assert(err, IsNil)
	}()
	readBuf := make([]byte, len(expected))
	_, err = f.ReadAt(readBuf, offset)
	c.Assert(err, IsNil)
	c.Assert(readBuf, DeepEquals, expected)
}

func writeDataToBlockDevice(ne *commonns.Executor, endpoint string, offsetInMB, dataCountInMB int64) error {
	return retry.Do(
		func() error {
			_, err := ne.Execute(
				nil,
				"dd",
				[]string{
					"if=/dev/urandom",
					fmt.Sprintf("of=%s", endpoint),
					"bs=1M",
					fmt.Sprintf("count=%d", dataCountInMB),
					fmt.Sprintf("seek=%d", offsetInMB),
					"status=none",
				},
				defaultTestExecuteTimeout,
			)
			return err
		},
		retry.Attempts(30),
		retry.Delay(1*time.Second),
		retry.DelayType(retry.FixedDelay),
		retry.LastErrorOnly(true),
		retry.OnRetry(func(n uint, err error) {
			logrus.WithFields(logrus.Fields{
				"attempt": n + 1,
				"error":   err,
			}).Warn("Write data to block device failed, retrying...")
		}),
	)
}
</file>

<file path="scripts/build">
#!/bin/bash
set -e

cd "$(dirname $0)"/..

go build ./...
</file>

<file path="scripts/ci">
#!/bin/bash
set -e

cd "$(dirname $0)"

./build
./validate
./test
</file>

<file path="scripts/entry">
#!/bin/bash
set -e

trap "chown -R $DAPPER_UID:$DAPPER_GID ." exit

export GOFLAGS=-mod=vendor

mkdir -p bin
if [ -e ./scripts/$1 ]; then
    ./scripts/"$*"
else
    "$@"
fi
</file>

<file path="scripts/test">
#!/bin/bash
set -e

LOCK_FILE="/tmp/longhorn-spdk.lock"
MAX_WAIT=36000  # seconds

# Try to acquire exclusive lock on $LOCK_FILE using file descriptor 200
# If the lock is held by another process, wait and retry
# Try to acquire exclusive lock on $LOCK_FILE using file descriptor 200
# Wait up to $MAX_WAIT seconds
exec 200>"$LOCK_FILE"
if ! flock -w "$MAX_WAIT" 200; then
  echo "Failed to acquire lock on $LOCK_FILE after $MAX_WAIT seconds. Exiting."
  exit 1
fi

cd "$(dirname $0)"/..

echo Running unit tests

# in case there is error before calling go test ...
touch coverage.out

# Check if hugepages are configured
hugepages="$(grep HugePages_Total < /proc/meminfo | awk '{print $2}')"
if [ -z "$hugepages" ] || [ 1 -gt $hugepages ]
then
  echo No hugepages configured on the host for the test
  exit 1
fi

mount --rbind /host/dev /dev
mount --rbind /host/sys /sys

echo "Checking /dev/hugepages"
if [ ! -d /dev/hugepages ]; then
  echo "Creating /dev/hugepages"
  mkdir -p /dev/hugepages
  mount -t hugetlbfs nodev /dev/hugepages
fi

trap "umount /dev && umount /sys && umount /dev/hugepages" EXIT

# Do cleanup first
losetup -D
trap "losetup -D" EXIT

PACKAGES="$(find . -name '*.go' -print0 | xargs -0 -I{} dirname {} |  cut -f2 -d/ | sort -u | grep -Ev '(^\.$|.git|.trash-cache|vendor|bin)' | sed -e 's!^!./!' -e 's!$!/...!')"

trap "rm -f /tmp/test-disk" EXIT

go test -v -p 1 -race -cover ${PACKAGES} -coverprofile=coverage.out -timeout 180m
</file>

<file path="scripts/validate">
#!/bin/bash
set -e

cd "$(dirname $0)"/..

echo Running go validation

PACKAGES="$(find . -name '*.go' -print0 | xargs -0 -I{} dirname {} |  cut -f2 -d/ | sort -u | grep -Ev '(^\.$|.git|.trash-cache|vendor|bin)' | sed -e 's!^!./!' -e 's!$!/...!')"
echo Packages: ${PACKAGES}

echo Running: go vet
go vet ${PACKAGES}

echo "Running: golangci-lint"
golangci-lint run --timeout=5m

echo Running: go fmt
test -z "$(go fmt ${PACKAGES} | tee /dev/stderr)"
</file>

<file path=".gitignore">
# Compiled Object files, Static and Dynamic libs (Shared Objects)
*.o
*.a
*.so

# Folders
_obj
_test

# Architecture specific extensions/prefixes
*.[568vq]
[568vq].out

*.cgo1.go
*.cgo2.c
_cgo_defun.c
_cgo_gotypes.go
_cgo_export.*

_testmain.go

*.exe
*.test
*.prof

coverage.out

# Rancher
.dapper
.dapper.tmp
trash.lock
.trash-conf
bin/
Dockerfile.dapper*

# patch tmp files
*.orig
*.rej

# ignores all goland project folders and files
.idea/
*.iml
*.ipr
</file>

<file path="go.mod">
module github.com/longhorn/longhorn-spdk-engine

go 1.25.3

require (
	github.com/0xPolygon/polygon-edge v1.3.3
	github.com/avast/retry-go/v4 v4.7.0
	github.com/cockroachdb/errors v1.12.0
	github.com/google/uuid v1.6.0
	github.com/jinzhu/copier v0.4.0
	github.com/longhorn/backupstore v0.0.0-20260329081928-dd6c86c9ba6d
	github.com/longhorn/go-common-libs v0.0.0-20260328134226-cafa38fc4ce8
	github.com/longhorn/go-spdk-helper v0.5.1-0.20260329081903-28b72c8abfa4
	github.com/longhorn/types v0.0.0-20260408014255-0ae353a2d888
	github.com/sirupsen/logrus v1.9.4
	go.uber.org/multierr v1.11.0
	google.golang.org/grpc v1.79.3
	google.golang.org/protobuf v1.36.11
	gopkg.in/check.v1 v1.0.0-20201130134442-10cb98267c6c
	k8s.io/apimachinery v0.35.3
	k8s.io/client-go v0.35.3
)

require (
	github.com/cockroachdb/logtags v0.0.0-20230118201751-21c54148d20b // indirect
	github.com/cockroachdb/redact v1.1.5 // indirect
	github.com/coreos/go-systemd/v22 v22.5.0 // indirect
	github.com/fxamacker/cbor/v2 v2.9.0 // indirect
	github.com/getsentry/sentry-go v0.27.0 // indirect
	github.com/godbus/dbus/v5 v5.1.0 // indirect
	github.com/gogo/protobuf v1.3.2 // indirect
	github.com/json-iterator/go v1.1.12 // indirect
	github.com/modern-go/concurrent v0.0.0-20180306012644-bacd9c7ef1dd // indirect
	github.com/modern-go/reflect2 v1.0.3-0.20250322232337-35a7c28c31ee // indirect
	github.com/munnerz/goautoneg v0.0.0-20191010083416-a7dc8b61c822 // indirect
	github.com/pkg/errors v0.9.1 // indirect
	github.com/x448/float16 v0.8.4 // indirect
	go.yaml.in/yaml/v2 v2.4.3 // indirect
	golang.org/x/net v0.49.0 // indirect
	gopkg.in/inf.v0 v0.9.1 // indirect
	k8s.io/kube-openapi v0.0.0-20250910181357-589584f1c912 // indirect
	sigs.k8s.io/json v0.0.0-20250730193827-2d320260d730 // indirect
	sigs.k8s.io/randfill v1.0.0 // indirect
	sigs.k8s.io/structured-merge-diff/v6 v6.3.0 // indirect
)

require (
	github.com/RoaringBitmap/roaring v1.9.4 // indirect
	github.com/beorn7/perks v1.0.1 // indirect
	github.com/bits-and-blooms/bitset v1.16.0 // indirect
	github.com/c9s/goprocinfo v0.0.0-20210130143923-c95fcf8c64a8 // indirect
	github.com/cespare/xxhash/v2 v2.3.0 // indirect
	github.com/gammazero/deque v1.0.0 // indirect
	github.com/gammazero/workerpool v1.1.3 // indirect
	github.com/go-logr/logr v1.4.3 // indirect
	github.com/go-ole/go-ole v1.3.0 // indirect
	github.com/kr/pretty v0.3.1 // indirect
	github.com/kr/text v0.2.0 // indirect
	github.com/mitchellh/go-ps v1.0.0 // indirect
	github.com/moby/sys/mountinfo v0.7.2 // indirect
	github.com/mschoch/smat v0.2.0 // indirect
	github.com/opencontainers/runc v1.1.14 // indirect
	github.com/opencontainers/runtime-spec v1.1.0 // indirect
	github.com/pierrec/lz4/v4 v4.1.26 // indirect
	github.com/power-devops/perfstat v0.0.0-20240221224432-82ca36839d55 // indirect
	github.com/prometheus/client_golang v1.20.5 // indirect
	github.com/prometheus/client_model v0.6.1 // indirect
	github.com/prometheus/common v0.60.1 // indirect
	github.com/prometheus/procfs v0.15.1 // indirect
	github.com/rogpeppe/go-internal v1.14.1 // indirect
	github.com/shirou/gopsutil/v3 v3.24.5 // indirect
	github.com/slok/goresilience v0.2.0 // indirect
	github.com/yusufpapurcu/wmi v1.2.4 // indirect
	golang.org/x/exp v0.0.0-20260312153236-7ab1446f8b90 // indirect
	golang.org/x/sys v0.40.0 // indirect
	golang.org/x/text v0.33.0 // indirect; sindirect
	google.golang.org/genproto/googleapis/rpc v0.0.0-20251202230838-ff82c1b0f217 // indirect
	k8s.io/klog/v2 v2.130.1 // indirect
	k8s.io/mount-utils v0.31.3 // indirect
	k8s.io/utils v0.0.0-20251002143259-bc988d571ff4 // indirect
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

<file path="Makefile">
TARGETS := $(shell ls scripts)
export SRC_BRANCH := master
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

trash: .dapper
	./.dapper -m bind trash

trash-keep: .dapper
	./.dapper -m bind trash -k

deps: trash

.DEFAULT_GOAL := ci

.PHONY: $(TARGETS)
</file>

<file path="README.md">
Longhorn SPDK Engine
==
[![Build Status](https://github.com/longhorn/longhorn-spdk-engine/actions/workflows/build.yml/badge.svg)](https://github.com/longhorn/longhorn-spdk-engine/actions/workflows/build.yml)[![Go Report Card](https://goreportcard.com/badge/github.com/longhorn/longhorn-spdk-engine)](https://goreportcard.com/report/github.com/longhorn/longhorn-spdk-engine)


Longhorn SPDK Engine is v2 data engine that integrates the performance and efficiency of SPDK (Storage Performance Development Kit) with the highly available, distributed block storage capabilities of Longhorn. By leveraging SPDK's user-space, poll-mode driver architecture, the engine achieves exceptional I/O throughput and low latency, catering to the demands of modern cloud-native workloads.


## Contribution

Please check [the main repo](https://github.com/longhorn/longhorn#community) for the contributing guide.

## License
Copyright (c) 2021-2024 The Longhorn Authors

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
</file>

<file path="renovate.json">
{
  "extends": ["github>longhorn/release:renovate-default"]
}
</file>

</files>
