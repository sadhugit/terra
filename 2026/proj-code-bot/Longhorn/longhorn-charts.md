This file is a merged representation of a subset of the codebase, containing files not matching ignore patterns, combined into a single document by Repomix.

<file_summary>
This section contains a summary of this file.

<purpose>
This file contains a packed representation of a subset of the repository's contents that is considered the most important context.
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
- Files matching these patterns are excluded: **/*_test.go, **/vendor/**, **/.git/**, **/bin/**
- Files matching patterns in .gitignore are excluded
- Files matching default ignore patterns are excluded
- Files are sorted by Git change count (files with more changes are at the bottom)
</notes>

</file_summary>

<directory_structure>
.github/
  workflows/
    release-ondemand.yml
charts/
  longhorn/
    templates/
      _helpers.tpl
      clusterrole.yaml
      clusterrolebinding.yaml
      crds.yaml
      daemonset-sa.yaml
      default-setting.yaml
      deployment-driver.yaml
      deployment-recovery-backend.yaml
      deployment-ui.yaml
      deployment-webhook.yaml
      ingress.yaml
      NOTES.txt
      postupgrade-job.yaml
      psp.yaml
      registry-secret.yaml
      serviceaccount.yaml
      services.yaml
      storageclass.yaml
      tls-secrets.yaml
      uninstall-job.yaml
    .helmignore
    app-readme.md
    Chart.yaml
    questions.yaml
    README.md
    values.yaml
LICENSE
README.md
</directory_structure>

<files>
This section contains the contents of the repository's files.

<file path=".github/workflows/release-ondemand.yml">
name: Release Charts on demand

on:
  workflow_dispatch:
    inputs:
      branch:
        description: 'Release branch'
        required: true

jobs:
  release:
    permissions:
      contents: write

    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v4
      with:
        ref: "${{ github.event.inputs.branch }}"

    - name: Fetch history
      run: git fetch --prune --unshallow

    - name: Install Helm
      uses: azure/setup-helm@v3

    - name: Configure Git
      run: |
        git config user.name "$GITHUB_ACTOR"
        git config user.email "$GITHUB_ACTOR@users.noreply.github.com"
        cat ./charts/longhorn/Chart.yaml

    - name: Run chart-releaser
      uses: helm/chart-releaser-action@v1.6.0
      env:
        CR_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
</file>

<file path="charts/longhorn/templates/_helpers.tpl">
{{/* vim: set filetype=mustache: */}}
{{/*
Expand the name of the chart.
*/}}
{{- define "longhorn.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" -}}
{{- end -}}

{{/*
Create a default fully qualified app name.
We truncate at 63 chars because some Kubernetes name fields are limited to this (by the DNS naming spec).
*/}}
{{- define "longhorn.fullname" -}}
{{- $name := default .Chart.Name .Values.nameOverride -}}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" -}}
{{- end -}}


{{- define "longhorn.managerIP" -}}
{{- $fullname := (include "longhorn.fullname" .) -}}
{{- printf "http://%s-backend:9500" $fullname | trunc 63 | trimSuffix "-" -}}
{{- end -}}


{{- define "secret" }}
{{- printf "{\"auths\": {\"%s\": {\"auth\": \"%s\"}}}" .Values.privateRegistry.registryUrl (printf "%s:%s" .Values.privateRegistry.registryUser .Values.privateRegistry.registryPasswd | b64enc) | b64enc }}
{{- end }}

{{- /*
longhorn.labels generates the standard Helm labels.
*/ -}}
{{- define "longhorn.labels" -}}
app.kubernetes.io/name: {{ template "longhorn.name" . }}
helm.sh/chart: {{ .Chart.Name }}-{{ .Chart.Version | replace "+" "_" }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/version: {{ .Chart.AppVersion }}
{{- end -}}


{{- define "system_default_registry" -}}
{{- if .Values.global.cattle.systemDefaultRegistry -}}
{{- printf "%s/" .Values.global.cattle.systemDefaultRegistry -}}
{{- else -}}
{{- "" -}}
{{- end -}}
{{- end -}}

{{- define "registry_url" -}}
{{- if .Values.privateRegistry.registryUrl -}}
{{- printf "%s/" .Values.privateRegistry.registryUrl -}}
{{- else -}}
{{ include "system_default_registry" . }}
{{- end -}}
{{- end -}}

{{- /*
 define the longhorn release namespace
*/ -}}
{{- define "release_namespace" -}}
{{- if .Values.namespaceOverride -}}
{{- .Values.namespaceOverride -}}
{{- else -}}
{{- .Release.Namespace -}}
{{- end -}}
{{- end -}}
</file>

<file path="charts/longhorn/templates/clusterrole.yaml">
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: longhorn-role
  labels: {{- include "longhorn.labels" . | nindent 4 }}
rules:
- apiGroups:
  - apiextensions.k8s.io
  resources:
  - customresourcedefinitions
  verbs:
  - "*"
- apiGroups: [""]
  resources: ["pods", "events", "persistentvolumes", "persistentvolumeclaims","persistentvolumeclaims/status", "nodes", "proxy/nodes", "pods/log", "secrets", "services", "endpoints", "configmaps", "serviceaccounts"]
  verbs: ["*"]
- apiGroups: [""]
  resources: ["namespaces"]
  verbs: ["get", "list"]
- apiGroups: ["apps"]
  resources: ["daemonsets", "statefulsets", "deployments"]
  verbs: ["*"]
- apiGroups: ["batch"]
  resources: ["jobs", "cronjobs"]
  verbs: ["*"]
- apiGroups: ["policy"]
  resources: ["poddisruptionbudgets", "podsecuritypolicies"]
  verbs: ["*"]
- apiGroups: ["scheduling.k8s.io"]
  resources: ["priorityclasses"]
  verbs: ["watch", "list"]
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses", "volumeattachments", "volumeattachments/status", "csinodes", "csidrivers"]
  verbs: ["*"]
- apiGroups: ["snapshot.storage.k8s.io"]
  resources: ["volumesnapshotclasses", "volumesnapshots", "volumesnapshotcontents", "volumesnapshotcontents/status"]
  verbs: ["*"]
- apiGroups: ["longhorn.io"]
  resources: ["volumes", "volumes/status", "engines", "engines/status", "replicas", "replicas/status", "settings",
              "engineimages", "engineimages/status", "nodes", "nodes/status", "instancemanagers", "instancemanagers/status",
              "sharemanagers", "sharemanagers/status", "backingimages", "backingimages/status",
              "backingimagemanagers", "backingimagemanagers/status", "backingimagedatasources", "backingimagedatasources/status",
              "backuptargets", "backuptargets/status", "backupvolumes", "backupvolumes/status", "backups", "backups/status",
              "recurringjobs", "recurringjobs/status", "orphans", "orphans/status", "snapshots", "snapshots/status",
              "supportbundles", "supportbundles/status", "systembackups", "systembackups/status", "systemrestores", "systemrestores/status"]
  verbs: ["*"]
- apiGroups: ["coordination.k8s.io"]
  resources: ["leases"]
  verbs: ["*"]
- apiGroups: ["metrics.k8s.io"]
  resources: ["pods", "nodes"]
  verbs: ["get", "list"]
- apiGroups: ["apiregistration.k8s.io"]
  resources: ["apiservices"]
  verbs: ["list", "watch"]
- apiGroups: ["admissionregistration.k8s.io"]
  resources: ["mutatingwebhookconfigurations", "validatingwebhookconfigurations"]
  verbs: ["get", "list", "create", "patch", "delete"]
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["roles", "rolebindings", "clusterrolebindings", "clusterroles"]
  verbs: ["*"]
</file>

<file path="charts/longhorn/templates/clusterrolebinding.yaml">
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: longhorn-bind
  labels: {{- include "longhorn.labels" . | nindent 4 }}
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: longhorn-role
subjects:
- kind: ServiceAccount
  name: longhorn-service-account
  namespace: {{ include "release_namespace" . }}
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: longhorn-support-bundle
  labels: {{- include "longhorn.labels" . | nindent 4 }}
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: longhorn-support-bundle
  namespace: {{ include "release_namespace" . }}
</file>

<file path="charts/longhorn/templates/crds.yaml">
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: backingimagedatasources.longhorn.io
spec:
  group: longhorn.io
  names:
    kind: BackingImageDataSource
    listKind: BackingImageDataSourceList
    plural: backingimagedatasources
    shortNames:
    - lhbids
    singular: backingimagedatasource
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The current state of the pod used to provision the backing image file from source
      jsonPath: .status.currentState
      name: State
      type: string
    - description: The data source type
      jsonPath: .spec.sourceType
      name: SourceType
      type: string
    - description: The node the backing image file will be prepared on
      jsonPath: .spec.nodeID
      name: Node
      type: string
    - description: The disk the backing image file will be prepared on
      jsonPath: .spec.diskUUID
      name: DiskUUID
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta1
    schema:
      openAPIV3Schema:
        description: BackingImageDataSource is where Longhorn stores backing image data source object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            x-kubernetes-preserve-unknown-fields: true
          status:
            x-kubernetes-preserve-unknown-fields: true
        type: object
    served: true
    storage: false
    subresources:
      status: {}
  - additionalPrinterColumns:
    - description: The system generated UUID of the provisioned backing image file
      jsonPath: .spec.uuid
      name: UUID
      type: string
    - description: The current state of the pod used to provision the backing image file from source
      jsonPath: .status.currentState
      name: State
      type: string
    - description: The data source type
      jsonPath: .spec.sourceType
      name: SourceType
      type: string
    - description: The backing image file size
      jsonPath: .status.size
      name: Size
      type: string
    - description: The node the backing image file will be prepared on
      jsonPath: .spec.nodeID
      name: Node
      type: string
    - description: The disk the backing image file will be prepared on
      jsonPath: .spec.diskUUID
      name: DiskUUID
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: BackingImageDataSource is where Longhorn stores backing image data source object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: BackingImageDataSourceSpec defines the desired state of the Longhorn backing image data source
            properties:
              checksum:
                type: string
              diskPath:
                type: string
              diskUUID:
                type: string
              fileTransferred:
                type: boolean
              nodeID:
                type: string
              parameters:
                additionalProperties:
                  type: string
                type: object
              sourceType:
                enum:
                - download
                - upload
                - export-from-volume
                type: string
              uuid:
                type: string
            type: object
          status:
            description: BackingImageDataSourceStatus defines the observed state of the Longhorn backing image data source
            properties:
              checksum:
                type: string
              currentState:
                type: string
              ip:
                type: string
              message:
                type: string
              ownerID:
                type: string
              progress:
                type: integer
              runningParameters:
                additionalProperties:
                  type: string
                nullable: true
                type: object
              size:
                format: int64
                type: integer
              storageIP:
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: backingimagemanagers.longhorn.io
spec:
  group: longhorn.io
  names:
    kind: BackingImageManager
    listKind: BackingImageManagerList
    plural: backingimagemanagers
    shortNames:
    - lhbim
    singular: backingimagemanager
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The current state of the manager
      jsonPath: .status.currentState
      name: State
      type: string
    - description: The image the manager pod will use
      jsonPath: .spec.image
      name: Image
      type: string
    - description: The node the manager is on
      jsonPath: .spec.nodeID
      name: Node
      type: string
    - description: The disk the manager is responsible for
      jsonPath: .spec.diskUUID
      name: DiskUUID
      type: string
    - description: The disk path the manager is using
      jsonPath: .spec.diskPath
      name: DiskPath
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta1
    schema:
      openAPIV3Schema:
        description: BackingImageManager is where Longhorn stores backing image manager object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            x-kubernetes-preserve-unknown-fields: true
          status:
            x-kubernetes-preserve-unknown-fields: true
        type: object
    served: true
    storage: false
    subresources:
      status: {}
  - additionalPrinterColumns:
    - description: The current state of the manager
      jsonPath: .status.currentState
      name: State
      type: string
    - description: The image the manager pod will use
      jsonPath: .spec.image
      name: Image
      type: string
    - description: The node the manager is on
      jsonPath: .spec.nodeID
      name: Node
      type: string
    - description: The disk the manager is responsible for
      jsonPath: .spec.diskUUID
      name: DiskUUID
      type: string
    - description: The disk path the manager is using
      jsonPath: .spec.diskPath
      name: DiskPath
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: BackingImageManager is where Longhorn stores backing image manager object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: BackingImageManagerSpec defines the desired state of the Longhorn backing image manager
            properties:
              backingImages:
                additionalProperties:
                  type: string
                type: object
              diskPath:
                type: string
              diskUUID:
                type: string
              image:
                type: string
              nodeID:
                type: string
            type: object
          status:
            description: BackingImageManagerStatus defines the observed state of the Longhorn backing image manager
            properties:
              apiMinVersion:
                type: integer
              apiVersion:
                type: integer
              backingImageFileMap:
                additionalProperties:
                  properties:
                    currentChecksum:
                      type: string
                    directory:
                      description: 'Deprecated: This field is useless.'
                      type: string
                    downloadProgress:
                      description: 'Deprecated: This field is renamed to `Progress`.'
                      type: integer
                    message:
                      type: string
                    name:
                      type: string
                    progress:
                      type: integer
                    senderManagerAddress:
                      type: string
                    sendingReference:
                      type: integer
                    size:
                      format: int64
                      type: integer
                    state:
                      type: string
                    url:
                      description: 'Deprecated: This field is useless now. The manager of backing image files doesn''t care if a file is downloaded and how.'
                      type: string
                    uuid:
                      type: string
                  type: object
                nullable: true
                type: object
              currentState:
                type: string
              ip:
                type: string
              ownerID:
                type: string
              storageIP:
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: backingimages.longhorn.io
spec:
  conversion:
    strategy: Webhook
    webhook:
      clientConfig:
        service:
          name: longhorn-conversion-webhook
          namespace: {{ include "release_namespace" . }}
          path: /v1/webhook/conversion
          port: 9443
      conversionReviewVersions:
      - v1beta2
      - v1beta1
  group: longhorn.io
  names:
    kind: BackingImage
    listKind: BackingImageList
    plural: backingimages
    shortNames:
    - lhbi
    singular: backingimage
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The backing image name
      jsonPath: .spec.image
      name: Image
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta1
    schema:
      openAPIV3Schema:
        description: BackingImage is where Longhorn stores backing image object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            x-kubernetes-preserve-unknown-fields: true
          status:
            x-kubernetes-preserve-unknown-fields: true
        type: object
    served: true
    storage: false
    subresources:
      status: {}
  - additionalPrinterColumns:
    - description: The system generated UUID
      jsonPath: .status.uuid
      name: UUID
      type: string
    - description: The source of the backing image file data
      jsonPath: .spec.sourceType
      name: SourceType
      type: string
    - description: The backing image file size in each disk
      jsonPath: .status.size
      name: Size
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: BackingImage is where Longhorn stores backing image object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: BackingImageSpec defines the desired state of the Longhorn backing image
            properties:
              checksum:
                type: string
              disks:
                additionalProperties:
                  type: string
                type: object
              imageURL:
                description: 'Deprecated: This kind of info will be included in the related BackingImageDataSource.'
                type: string
              sourceParameters:
                additionalProperties:
                  type: string
                type: object
              sourceType:
                enum:
                - download
                - upload
                - export-from-volume
                type: string
            type: object
          status:
            description: BackingImageStatus defines the observed state of the Longhorn backing image status
            properties:
              checksum:
                type: string
              diskDownloadProgressMap:
                additionalProperties:
                  type: integer
                description: 'Deprecated: Replaced by field `Progress` in `DiskFileStatusMap`.'
                nullable: true
                type: object
              diskDownloadStateMap:
                additionalProperties:
                  description: BackingImageDownloadState is replaced by BackingImageState.
                  type: string
                description: 'Deprecated: Replaced by field `State` in `DiskFileStatusMap`.'
                nullable: true
                type: object
              diskFileStatusMap:
                additionalProperties:
                  properties:
                    lastStateTransitionTime:
                      type: string
                    message:
                      type: string
                    progress:
                      type: integer
                    state:
                      type: string
                  type: object
                nullable: true
                type: object
              diskLastRefAtMap:
                additionalProperties:
                  type: string
                nullable: true
                type: object
              ownerID:
                type: string
              size:
                format: int64
                type: integer
              uuid:
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: backups.longhorn.io
spec:
  group: longhorn.io
  names:
    kind: Backup
    listKind: BackupList
    plural: backups
    shortNames:
    - lhb
    singular: backup
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The snapshot name
      jsonPath: .status.snapshotName
      name: SnapshotName
      type: string
    - description: The snapshot size
      jsonPath: .status.size
      name: SnapshotSize
      type: string
    - description: The snapshot creation time
      jsonPath: .status.snapshotCreatedAt
      name: SnapshotCreatedAt
      type: string
    - description: The backup state
      jsonPath: .status.state
      name: State
      type: string
    - description: The backup last synced time
      jsonPath: .status.lastSyncedAt
      name: LastSyncedAt
      type: string
    name: v1beta1
    schema:
      openAPIV3Schema:
        description: Backup is where Longhorn stores backup object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            x-kubernetes-preserve-unknown-fields: true
          status:
            x-kubernetes-preserve-unknown-fields: true
        type: object
    served: true
    storage: false
    subresources:
      status: {}
  - additionalPrinterColumns:
    - description: The snapshot name
      jsonPath: .status.snapshotName
      name: SnapshotName
      type: string
    - description: The snapshot size
      jsonPath: .status.size
      name: SnapshotSize
      type: string
    - description: The snapshot creation time
      jsonPath: .status.snapshotCreatedAt
      name: SnapshotCreatedAt
      type: string
    - description: The backup state
      jsonPath: .status.state
      name: State
      type: string
    - description: The backup last synced time
      jsonPath: .status.lastSyncedAt
      name: LastSyncedAt
      type: string
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: Backup is where Longhorn stores backup object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: BackupSpec defines the desired state of the Longhorn backup
            properties:
              labels:
                additionalProperties:
                  type: string
                description: The labels of snapshot backup.
                type: object
              snapshotName:
                description: The snapshot name.
                type: string
              syncRequestedAt:
                description: The time to request run sync the remote backup.
                format: date-time
                nullable: true
                type: string
            type: object
          status:
            description: BackupStatus defines the observed state of the Longhorn backup
            properties:
              backupCreatedAt:
                description: The snapshot backup upload finished time.
                type: string
              error:
                description: The error message when taking the snapshot backup.
                type: string
              labels:
                additionalProperties:
                  type: string
                description: The labels of snapshot backup.
                nullable: true
                type: object
              lastSyncedAt:
                description: The last time that the backup was synced with the remote backup target.
                format: date-time
                nullable: true
                type: string
              messages:
                additionalProperties:
                  type: string
                description: The error messages when calling longhorn engine on listing or inspecting backups.
                nullable: true
                type: object
              ownerID:
                description: The node ID on which the controller is responsible to reconcile this backup CR.
                type: string
              progress:
                description: The snapshot backup progress.
                type: integer
              replicaAddress:
                description: The address of the replica that runs snapshot backup.
                type: string
              size:
                description: The snapshot size.
                type: string
              snapshotCreatedAt:
                description: The snapshot creation time.
                type: string
              snapshotName:
                description: The snapshot name.
                type: string
              state:
                description: The backup creation state. Can be "", "InProgress", "Completed", "Error", "Unknown".
                type: string
              url:
                description: The snapshot backup URL.
                type: string
              volumeBackingImageName:
                description: The volume's backing image name.
                type: string
              volumeCreated:
                description: The volume creation time.
                type: string
              volumeName:
                description: The volume name.
                type: string
              volumeSize:
                description: The volume size.
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: backuptargets.longhorn.io
spec:
  conversion:
    strategy: Webhook
    webhook:
      clientConfig:
        service:
          name: longhorn-conversion-webhook
          namespace: {{ include "release_namespace" . }}
          path: /v1/webhook/conversion
          port: 9443
      conversionReviewVersions:
      - v1beta2
      - v1beta1
  group: longhorn.io
  names:
    kind: BackupTarget
    listKind: BackupTargetList
    plural: backuptargets
    shortNames:
    - lhbt
    singular: backuptarget
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The backup target URL
      jsonPath: .spec.backupTargetURL
      name: URL
      type: string
    - description: The backup target credential secret
      jsonPath: .spec.credentialSecret
      name: Credential
      type: string
    - description: The backup target poll interval
      jsonPath: .spec.pollInterval
      name: LastBackupAt
      type: string
    - description: Indicate whether the backup target is available or not
      jsonPath: .status.available
      name: Available
      type: boolean
    - description: The backup target last synced time
      jsonPath: .status.lastSyncedAt
      name: LastSyncedAt
      type: string
    name: v1beta1
    schema:
      openAPIV3Schema:
        description: BackupTarget is where Longhorn stores backup target object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            x-kubernetes-preserve-unknown-fields: true
          status:
            x-kubernetes-preserve-unknown-fields: true
        type: object
    served: true
    storage: false
    subresources:
      status: {}
  - additionalPrinterColumns:
    - description: The backup target URL
      jsonPath: .spec.backupTargetURL
      name: URL
      type: string
    - description: The backup target credential secret
      jsonPath: .spec.credentialSecret
      name: Credential
      type: string
    - description: The backup target poll interval
      jsonPath: .spec.pollInterval
      name: LastBackupAt
      type: string
    - description: Indicate whether the backup target is available or not
      jsonPath: .status.available
      name: Available
      type: boolean
    - description: The backup target last synced time
      jsonPath: .status.lastSyncedAt
      name: LastSyncedAt
      type: string
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: BackupTarget is where Longhorn stores backup target object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: BackupTargetSpec defines the desired state of the Longhorn backup target
            properties:
              backupTargetURL:
                description: The backup target URL.
                type: string
              credentialSecret:
                description: The backup target credential secret.
                type: string
              pollInterval:
                description: The interval that the cluster needs to run sync with the backup target.
                type: string
              syncRequestedAt:
                description: The time to request run sync the remote backup target.
                format: date-time
                nullable: true
                type: string
            type: object
          status:
            description: BackupTargetStatus defines the observed state of the Longhorn backup target
            properties:
              available:
                description: Available indicates if the remote backup target is available or not.
                type: boolean
              conditions:
                description: Records the reason on why the backup target is unavailable.
                items:
                  properties:
                    lastProbeTime:
                      description: Last time we probed the condition.
                      type: string
                    lastTransitionTime:
                      description: Last time the condition transitioned from one status to another.
                      type: string
                    message:
                      description: Human-readable message indicating details about last transition.
                      type: string
                    reason:
                      description: Unique, one-word, CamelCase reason for the condition's last transition.
                      type: string
                    status:
                      description: Status is the status of the condition. Can be True, False, Unknown.
                      type: string
                    type:
                      description: Type is the type of the condition.
                      type: string
                  type: object
                nullable: true
                type: array
              lastSyncedAt:
                description: The last time that the controller synced with the remote backup target.
                format: date-time
                nullable: true
                type: string
              ownerID:
                description: The node ID on which the controller is responsible to reconcile this backup target CR.
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: backupvolumes.longhorn.io
spec:
  group: longhorn.io
  names:
    kind: BackupVolume
    listKind: BackupVolumeList
    plural: backupvolumes
    shortNames:
    - lhbv
    singular: backupvolume
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The backup volume creation time
      jsonPath: .status.createdAt
      name: CreatedAt
      type: string
    - description: The backup volume last backup name
      jsonPath: .status.lastBackupName
      name: LastBackupName
      type: string
    - description: The backup volume last backup time
      jsonPath: .status.lastBackupAt
      name: LastBackupAt
      type: string
    - description: The backup volume last synced time
      jsonPath: .status.lastSyncedAt
      name: LastSyncedAt
      type: string
    name: v1beta1
    schema:
      openAPIV3Schema:
        description: BackupVolume is where Longhorn stores backup volume object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            x-kubernetes-preserve-unknown-fields: true
          status:
            x-kubernetes-preserve-unknown-fields: true
        type: object
    served: true
    storage: false
    subresources:
      status: {}
  - additionalPrinterColumns:
    - description: The backup volume creation time
      jsonPath: .status.createdAt
      name: CreatedAt
      type: string
    - description: The backup volume last backup name
      jsonPath: .status.lastBackupName
      name: LastBackupName
      type: string
    - description: The backup volume last backup time
      jsonPath: .status.lastBackupAt
      name: LastBackupAt
      type: string
    - description: The backup volume last synced time
      jsonPath: .status.lastSyncedAt
      name: LastSyncedAt
      type: string
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: BackupVolume is where Longhorn stores backup volume object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: BackupVolumeSpec defines the desired state of the Longhorn backup volume
            properties:
              syncRequestedAt:
                description: The time to request run sync the remote backup volume.
                format: date-time
                nullable: true
                type: string
            type: object
          status:
            description: BackupVolumeStatus defines the observed state of the Longhorn backup volume
            properties:
              backingImageChecksum:
                description: the backing image checksum.
                type: string
              backingImageName:
                description: The backing image name.
                type: string
              createdAt:
                description: The backup volume creation time.
                type: string
              dataStored:
                description: The backup volume block count.
                type: string
              labels:
                additionalProperties:
                  type: string
                description: The backup volume labels.
                nullable: true
                type: object
              lastBackupAt:
                description: The latest volume backup time.
                type: string
              lastBackupName:
                description: The latest volume backup name.
                type: string
              lastModificationTime:
                description: The backup volume config last modification time.
                format: date-time
                nullable: true
                type: string
              lastSyncedAt:
                description: The last time that the backup volume was synced into the cluster.
                format: date-time
                nullable: true
                type: string
              messages:
                additionalProperties:
                  type: string
                description: The error messages when call longhorn engine on list or inspect backup volumes.
                nullable: true
                type: object
              ownerID:
                description: The node ID on which the controller is responsible to reconcile this backup volume CR.
                type: string
              size:
                description: The backup volume size.
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: engineimages.longhorn.io
spec:
  preserveUnknownFields: false
  conversion:
    strategy: Webhook
    webhook:
      clientConfig:
        service:
          name: longhorn-conversion-webhook
          namespace: {{ include "release_namespace" . }}
          path: /v1/webhook/conversion
          port: 9443
      conversionReviewVersions:
      - v1beta2
      - v1beta1
  group: longhorn.io
  names:
    kind: EngineImage
    listKind: EngineImageList
    plural: engineimages
    shortNames:
    - lhei
    singular: engineimage
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: State of the engine image
      jsonPath: .status.state
      name: State
      type: string
    - description: The Longhorn engine image
      jsonPath: .spec.image
      name: Image
      type: string
    - description: Number of resources using the engine image
      jsonPath: .status.refCount
      name: RefCount
      type: integer
    - description: The build date of the engine image
      jsonPath: .status.buildDate
      name: BuildDate
      type: date
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta1
    schema:
      openAPIV3Schema:
        description: EngineImage is where Longhorn stores engine image object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            x-kubernetes-preserve-unknown-fields: true
          status:
            x-kubernetes-preserve-unknown-fields: true
        type: object
    served: true
    storage: false
    subresources:
      status: {}
  - additionalPrinterColumns:
    - description: State of the engine image
      jsonPath: .status.state
      name: State
      type: string
    - description: The Longhorn engine image
      jsonPath: .spec.image
      name: Image
      type: string
    - description: Number of resources using the engine image
      jsonPath: .status.refCount
      name: RefCount
      type: integer
    - description: The build date of the engine image
      jsonPath: .status.buildDate
      name: BuildDate
      type: date
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: EngineImage is where Longhorn stores engine image object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: EngineImageSpec defines the desired state of the Longhorn engine image
            properties:
              image:
                minLength: 1
                type: string
            required:
            - image
            type: object
          status:
            description: EngineImageStatus defines the observed state of the Longhorn engine image
            properties:
              buildDate:
                type: string
              cliAPIMinVersion:
                type: integer
              cliAPIVersion:
                type: integer
              conditions:
                items:
                  properties:
                    lastProbeTime:
                      description: Last time we probed the condition.
                      type: string
                    lastTransitionTime:
                      description: Last time the condition transitioned from one status to another.
                      type: string
                    message:
                      description: Human-readable message indicating details about last transition.
                      type: string
                    reason:
                      description: Unique, one-word, CamelCase reason for the condition's last transition.
                      type: string
                    status:
                      description: Status is the status of the condition. Can be True, False, Unknown.
                      type: string
                    type:
                      description: Type is the type of the condition.
                      type: string
                  type: object
                nullable: true
                type: array
              controllerAPIMinVersion:
                type: integer
              controllerAPIVersion:
                type: integer
              dataFormatMinVersion:
                type: integer
              dataFormatVersion:
                type: integer
              gitCommit:
                type: string
              noRefSince:
                type: string
              nodeDeploymentMap:
                additionalProperties:
                  type: boolean
                nullable: true
                type: object
              ownerID:
                type: string
              refCount:
                type: integer
              state:
                type: string
              version:
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: engines.longhorn.io
spec:
  group: longhorn.io
  names:
    kind: Engine
    listKind: EngineList
    plural: engines
    shortNames:
    - lhe
    singular: engine
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The current state of the engine
      jsonPath: .status.currentState
      name: State
      type: string
    - description: The node that the engine is on
      jsonPath: .spec.nodeID
      name: Node
      type: string
    - description: The instance manager of the engine
      jsonPath: .status.instanceManagerName
      name: InstanceManager
      type: string
    - description: The current image of the engine
      jsonPath: .status.currentImage
      name: Image
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta1
    schema:
      openAPIV3Schema:
        description: Engine is where Longhorn stores engine object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            x-kubernetes-preserve-unknown-fields: true
          status:
            x-kubernetes-preserve-unknown-fields: true
        type: object
    served: true
    storage: false
    subresources:
      status: {}
  - additionalPrinterColumns:
    - description: The current state of the engine
      jsonPath: .status.currentState
      name: State
      type: string
    - description: The node that the engine is on
      jsonPath: .spec.nodeID
      name: Node
      type: string
    - description: The instance manager of the engine
      jsonPath: .status.instanceManagerName
      name: InstanceManager
      type: string
    - description: The current image of the engine
      jsonPath: .status.currentImage
      name: Image
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: Engine is where Longhorn stores engine object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: EngineSpec defines the desired state of the Longhorn engine
            properties:
              active:
                type: boolean
              backupVolume:
                type: string
              desireState:
                type: string
              disableFrontend:
                type: boolean
              engineImage:
                type: string
              frontend:
                enum:
                - blockdev
                - iscsi
                - ""
                type: string
              logRequested:
                type: boolean
              nodeID:
                type: string
              replicaAddressMap:
                additionalProperties:
                  type: string
                type: object
              requestedBackupRestore:
                type: string
              requestedDataSource:
                type: string
              revisionCounterDisabled:
                type: boolean
              salvageRequested:
                type: boolean
              unmapMarkSnapChainRemovedEnabled:
                type: boolean
              upgradedReplicaAddressMap:
                additionalProperties:
                  type: string
                type: object
              volumeName:
                type: string
              volumeSize:
                format: int64
                type: string
            type: object
          status:
            description: EngineStatus defines the observed state of the Longhorn engine
            properties:
              backupStatus:
                additionalProperties:
                  properties:
                    backupURL:
                      type: string
                    error:
                      type: string
                    progress:
                      type: integer
                    replicaAddress:
                      type: string
                    snapshotName:
                      type: string
                    state:
                      type: string
                  type: object
                nullable: true
                type: object
              cloneStatus:
                additionalProperties:
                  properties:
                    error:
                      type: string
                    fromReplicaAddress:
                      type: string
                    isCloning:
                      type: boolean
                    progress:
                      type: integer
                    snapshotName:
                      type: string
                    state:
                      type: string
                  type: object
                nullable: true
                type: object
              conditions:
                items:
                  properties:
                    lastProbeTime:
                      description: Last time we probed the condition.
                      type: string
                    lastTransitionTime:
                      description: Last time the condition transitioned from one status to another.
                      type: string
                    message:
                      description: Human-readable message indicating details about last transition.
                      type: string
                    reason:
                      description: Unique, one-word, CamelCase reason for the condition's last transition.
                      type: string
                    status:
                      description: Status is the status of the condition. Can be True, False, Unknown.
                      type: string
                    type:
                      description: Type is the type of the condition.
                      type: string
                  type: object
                nullable: true
                type: array
              currentImage:
                type: string
              currentReplicaAddressMap:
                additionalProperties:
                  type: string
                nullable: true
                type: object
              currentSize:
                format: int64
                type: string
              currentState:
                type: string
              endpoint:
                type: string
              instanceManagerName:
                type: string
              ip:
                type: string
              isExpanding:
                type: boolean
              lastExpansionError:
                type: string
              lastExpansionFailedAt:
                type: string
              lastRestoredBackup:
                type: string
              logFetched:
                type: boolean
              ownerID:
                type: string
              port:
                type: integer
              purgeStatus:
                additionalProperties:
                  properties:
                    error:
                      type: string
                    isPurging:
                      type: boolean
                    progress:
                      type: integer
                    state:
                      type: string
                  type: object
                nullable: true
                type: object
              rebuildStatus:
                additionalProperties:
                  properties:
                    error:
                      type: string
                    fromReplicaAddress:
                      type: string
                    isRebuilding:
                      type: boolean
                    progress:
                      type: integer
                    state:
                      type: string
                  type: object
                nullable: true
                type: object
              replicaModeMap:
                additionalProperties:
                  type: string
                nullable: true
                type: object
              restoreStatus:
                additionalProperties:
                  properties:
                    backupURL:
                      type: string
                    currentRestoringBackup:
                      type: string
                    error:
                      type: string
                    filename:
                      type: string
                    isRestoring:
                      type: boolean
                    lastRestored:
                      type: string
                    progress:
                      type: integer
                    state:
                      type: string
                  type: object
                nullable: true
                type: object
              salvageExecuted:
                type: boolean
              snapshots:
                additionalProperties:
                  properties:
                    children:
                      additionalProperties:
                        type: boolean
                      nullable: true
                      type: object
                    created:
                      type: string
                    labels:
                      additionalProperties:
                        type: string
                      nullable: true
                      type: object
                    name:
                      type: string
                    parent:
                      type: string
                    removed:
                      type: boolean
                    size:
                      type: string
                    usercreated:
                      type: boolean
                  type: object
                nullable: true
                type: object
              snapshotsError:
                type: string
              started:
                type: boolean
              storageIP:
                type: string
              unmapMarkSnapChainRemovedEnabled:
                type: boolean
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: instancemanagers.longhorn.io
spec:
  group: longhorn.io
  names:
    kind: InstanceManager
    listKind: InstanceManagerList
    plural: instancemanagers
    shortNames:
    - lhim
    singular: instancemanager
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The state of the instance manager
      jsonPath: .status.currentState
      name: State
      type: string
    - description: The type of the instance manager (engine or replica)
      jsonPath: .spec.type
      name: Type
      type: string
    - description: The node that the instance manager is running on
      jsonPath: .spec.nodeID
      name: Node
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta1
    schema:
      openAPIV3Schema:
        description: InstanceManager is where Longhorn stores instance manager object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            x-kubernetes-preserve-unknown-fields: true
          status:
            x-kubernetes-preserve-unknown-fields: true
        type: object
    served: true
    storage: false
    subresources:
      status: {}
  - additionalPrinterColumns:
    - description: The state of the instance manager
      jsonPath: .status.currentState
      name: State
      type: string
    - description: The type of the instance manager (engine or replica)
      jsonPath: .spec.type
      name: Type
      type: string
    - description: The node that the instance manager is running on
      jsonPath: .spec.nodeID
      name: Node
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: InstanceManager is where Longhorn stores instance manager object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: InstanceManagerSpec defines the desired state of the Longhorn instancer manager
            properties:
              engineImage:
                description: 'TODO: deprecate this field'
                type: string
              image:
                type: string
              nodeID:
                type: string
              type:
                enum:
                - engine
                - replica
                type: string
            type: object
          status:
            description: InstanceManagerStatus defines the observed state of the Longhorn instance manager
            properties:
              apiMinVersion:
                type: integer
              apiVersion:
                type: integer
              proxyApiMinVersion:
                type: integer
              proxyApiVersion:
                type: integer
              currentState:
                type: string
              instances:
                additionalProperties:
                  properties:
                    spec:
                      properties:
                        name:
                          type: string
                      type: object
                    status:
                      properties:
                        endpoint:
                          type: string
                        errorMsg:
                          type: string
                        listen:
                          type: string
                        portEnd:
                          format: int32
                          type: integer
                        portStart:
                          format: int32
                          type: integer
                        resourceVersion:
                          format: int64
                          type: integer
                        state:
                          type: string
                        type:
                          type: string
                      type: object
                  type: object
                nullable: true
                type: object
              ip:
                type: string
              ownerID:
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: nodes.longhorn.io
spec:
  preserveUnknownFields: false
  conversion:
    strategy: Webhook
    webhook:
      clientConfig:
        service:
          name: longhorn-conversion-webhook
          namespace: {{ include "release_namespace" . }}
          path: /v1/webhook/conversion
          port: 9443
      conversionReviewVersions:
      - v1beta2
      - v1beta1
  group: longhorn.io
  names:
    kind: Node
    listKind: NodeList
    plural: nodes
    shortNames:
    - lhn
    singular: node
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: Indicate whether the node is ready
      jsonPath: .status.conditions['Ready']['status']
      name: Ready
      type: string
    - description: Indicate whether the user disabled/enabled replica scheduling for the node
      jsonPath: .spec.allowScheduling
      name: AllowScheduling
      type: boolean
    - description: Indicate whether Longhorn can schedule replicas on the node
      jsonPath: .status.conditions['Schedulable']['status']
      name: Schedulable
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta1
    schema:
      openAPIV3Schema:
        description: Node is where Longhorn stores Longhorn node object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            x-kubernetes-preserve-unknown-fields: true
          status:
            x-kubernetes-preserve-unknown-fields: true
        type: object
    served: true
    storage: false
    subresources:
      status: {}
  - additionalPrinterColumns:
    - description: Indicate whether the node is ready
      jsonPath: .status.conditions[?(@.type=='Ready')].status
      name: Ready
      type: string
    - description: Indicate whether the user disabled/enabled replica scheduling for the node
      jsonPath: .spec.allowScheduling
      name: AllowScheduling
      type: boolean
    - description: Indicate whether Longhorn can schedule replicas on the node
      jsonPath: .status.conditions[?(@.type=='Schedulable')].status
      name: Schedulable
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: Node is where Longhorn stores Longhorn node object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: NodeSpec defines the desired state of the Longhorn node
            properties:
              allowScheduling:
                type: boolean
              disks:
                additionalProperties:
                  properties:
                    allowScheduling:
                      type: boolean
                    evictionRequested:
                      type: boolean
                    path:
                      type: string
                    storageReserved:
                      format: int64
                      type: integer
                    tags:
                      items:
                        type: string
                      type: array
                  type: object
                type: object
              engineManagerCPURequest:
                type: integer
              evictionRequested:
                type: boolean
              name:
                type: string
              replicaManagerCPURequest:
                type: integer
              tags:
                items:
                  type: string
                type: array
            type: object
          status:
            description: NodeStatus defines the observed state of the Longhorn node
            properties:
              conditions:
                items:
                  properties:
                    lastProbeTime:
                      description: Last time we probed the condition.
                      type: string
                    lastTransitionTime:
                      description: Last time the condition transitioned from one status to another.
                      type: string
                    message:
                      description: Human-readable message indicating details about last transition.
                      type: string
                    reason:
                      description: Unique, one-word, CamelCase reason for the condition's last transition.
                      type: string
                    status:
                      description: Status is the status of the condition. Can be True, False, Unknown.
                      type: string
                    type:
                      description: Type is the type of the condition.
                      type: string
                  type: object
                nullable: true
                type: array
              diskStatus:
                additionalProperties:
                  properties:
                    conditions:
                      items:
                        properties:
                          lastProbeTime:
                            description: Last time we probed the condition.
                            type: string
                          lastTransitionTime:
                            description: Last time the condition transitioned from one status to another.
                            type: string
                          message:
                            description: Human-readable message indicating details about last transition.
                            type: string
                          reason:
                            description: Unique, one-word, CamelCase reason for the condition's last transition.
                            type: string
                          status:
                            description: Status is the status of the condition. Can be True, False, Unknown.
                            type: string
                          type:
                            description: Type is the type of the condition.
                            type: string
                        type: object
                      nullable: true
                      type: array
                    diskUUID:
                      type: string
                    scheduledReplica:
                      additionalProperties:
                        format: int64
                        type: integer
                      nullable: true
                      type: object
                    storageAvailable:
                      format: int64
                      type: integer
                    storageMaximum:
                      format: int64
                      type: integer
                    storageScheduled:
                      format: int64
                      type: integer
                  type: object
                nullable: true
                type: object
              region:
                type: string
              snapshotCheckStatus:
                properties:
                  lastPeriodicCheckedAt:
                    format: date-time
                    type: string
                  snapshotCheckState:
                    type: string
                type: object
              zone:
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: orphans.longhorn.io
spec:
  group: longhorn.io
  names:
    kind: Orphan
    listKind: OrphanList
    plural: orphans
    shortNames:
    - lho
    singular: orphan
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The type of the orphan
      jsonPath: .spec.orphanType
      name: Type
      type: string
    - description: The node that the orphan is on
      jsonPath: .spec.nodeID
      name: Node
      type: string
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: Orphan is where Longhorn stores orphan object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: OrphanSpec defines the desired state of the Longhorn orphaned data
            properties:
              nodeID:
                description: The node ID on which the controller is responsible to reconcile this orphan CR.
                type: string
              orphanType:
                description: The type of the orphaned data. Can be "replica".
                type: string
              parameters:
                additionalProperties:
                  type: string
                description: The parameters of the orphaned data
                type: object
            type: object
          status:
            description: OrphanStatus defines the observed state of the Longhorn orphaned data
            properties:
              conditions:
                items:
                  properties:
                    lastProbeTime:
                      description: Last time we probed the condition.
                      type: string
                    lastTransitionTime:
                      description: Last time the condition transitioned from one status to another.
                      type: string
                    message:
                      description: Human-readable message indicating details about last transition.
                      type: string
                    reason:
                      description: Unique, one-word, CamelCase reason for the condition's last transition.
                      type: string
                    status:
                      description: Status is the status of the condition. Can be True, False, Unknown.
                      type: string
                    type:
                      description: Type is the type of the condition.
                      type: string
                  type: object
                nullable: true
                type: array
              ownerID:
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels:
    longhorn-manager: ""
  name: recurringjobs.longhorn.io
spec:
  group: longhorn.io
  names:
    kind: RecurringJob
    listKind: RecurringJobList
    plural: recurringjobs
    shortNames:
    - lhrj
    singular: recurringjob
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: Sets groupings to the jobs. When set to "default" group will be added to the volume label when no other job label exist in volume
      jsonPath: .spec.groups
      name: Groups
      type: string
    - description: Should be one of "backup" or "snapshot"
      jsonPath: .spec.task
      name: Task
      type: string
    - description: The cron expression represents recurring job scheduling
      jsonPath: .spec.cron
      name: Cron
      type: string
    - description: The number of snapshots/backups to keep for the volume
      jsonPath: .spec.retain
      name: Retain
      type: integer
    - description: The concurrent job to run by each cron job
      jsonPath: .spec.concurrency
      name: Concurrency
      type: integer
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    - description: Specify the labels
      jsonPath: .spec.labels
      name: Labels
      type: string
    name: v1beta1
    schema:
      openAPIV3Schema:
        description: RecurringJob is where Longhorn stores recurring job object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            x-kubernetes-preserve-unknown-fields: true
          status:
            x-kubernetes-preserve-unknown-fields: true
        type: object
    served: true
    storage: false
    subresources:
      status: {}
  - additionalPrinterColumns:
    - description: Sets groupings to the jobs. When set to "default" group will be added to the volume label when no other job label exist in volume
      jsonPath: .spec.groups
      name: Groups
      type: string
    - description: Should be one of "backup" or "snapshot"
      jsonPath: .spec.task
      name: Task
      type: string
    - description: The cron expression represents recurring job scheduling
      jsonPath: .spec.cron
      name: Cron
      type: string
    - description: The number of snapshots/backups to keep for the volume
      jsonPath: .spec.retain
      name: Retain
      type: integer
    - description: The concurrent job to run by each cron job
      jsonPath: .spec.concurrency
      name: Concurrency
      type: integer
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    - description: Specify the labels
      jsonPath: .spec.labels
      name: Labels
      type: string
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: RecurringJob is where Longhorn stores recurring job object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: RecurringJobSpec defines the desired state of the Longhorn recurring job
            properties:
              concurrency:
                description: The concurrency of taking the snapshot/backup.
                type: integer
              cron:
                description: The cron setting.
                type: string
              groups:
                description: The recurring job group.
                items:
                  type: string
                type: array
              labels:
                additionalProperties:
                  type: string
                description: The label of the snapshot/backup.
                type: object
              name:
                description: The recurring job name.
                type: string
              retain:
                description: The retain count of the snapshot/backup.
                type: integer
              task:
                description: The recurring job type. Can be "snapshot" or "backup".
                enum:
                - snapshot
                - backup
                type: string
            type: object
          status:
            description: RecurringJobStatus defines the observed state of the Longhorn recurring job
            properties:
              ownerID:
                description: The owner ID which is responsible to reconcile this recurring job CR.
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: replicas.longhorn.io
spec:
  group: longhorn.io
  names:
    kind: Replica
    listKind: ReplicaList
    plural: replicas
    shortNames:
    - lhr
    singular: replica
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The current state of the replica
      jsonPath: .status.currentState
      name: State
      type: string
    - description: The node that the replica is on
      jsonPath: .spec.nodeID
      name: Node
      type: string
    - description: The disk that the replica is on
      jsonPath: .spec.diskID
      name: Disk
      type: string
    - description: The instance manager of the replica
      jsonPath: .status.instanceManagerName
      name: InstanceManager
      type: string
    - description: The current image of the replica
      jsonPath: .status.currentImage
      name: Image
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta1
    schema:
      openAPIV3Schema:
        description: Replica is where Longhorn stores replica object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            x-kubernetes-preserve-unknown-fields: true
          status:
            x-kubernetes-preserve-unknown-fields: true
        type: object
    served: true
    storage: false
    subresources:
      status: {}
  - additionalPrinterColumns:
    - description: The current state of the replica
      jsonPath: .status.currentState
      name: State
      type: string
    - description: The node that the replica is on
      jsonPath: .spec.nodeID
      name: Node
      type: string
    - description: The disk that the replica is on
      jsonPath: .spec.diskID
      name: Disk
      type: string
    - description: The instance manager of the replica
      jsonPath: .status.instanceManagerName
      name: InstanceManager
      type: string
    - description: The current image of the replica
      jsonPath: .status.currentImage
      name: Image
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: Replica is where Longhorn stores replica object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: ReplicaSpec defines the desired state of the Longhorn replica
            properties:
              active:
                type: boolean
              backingImage:
                type: string
              baseImage:
                description: Deprecated. Rename to BackingImage
                type: string
              dataDirectoryName:
                type: string
              dataPath:
                description: Deprecated
                type: string
              desireState:
                type: string
              diskID:
                type: string
              diskPath:
                type: string
              engineImage:
                type: string
              engineName:
                type: string
              failedAt:
                type: string
              hardNodeAffinity:
                type: string
              healthyAt:
                type: string
              logRequested:
                type: boolean
              nodeID:
                type: string
              rebuildRetryCount:
                type: integer
              revisionCounterDisabled:
                type: boolean
              salvageRequested:
                type: boolean
              unmapMarkDiskChainRemovedEnabled:
                type: boolean
              volumeName:
                type: string
              volumeSize:
                format: int64
                type: string
            type: object
          status:
            description: ReplicaStatus defines the observed state of the Longhorn replica
            properties:
              conditions:
                items:
                  properties:
                    lastProbeTime:
                      description: Last time we probed the condition.
                      type: string
                    lastTransitionTime:
                      description: Last time the condition transitioned from one status to another.
                      type: string
                    message:
                      description: Human-readable message indicating details about last transition.
                      type: string
                    reason:
                      description: Unique, one-word, CamelCase reason for the condition's last transition.
                      type: string
                    status:
                      description: Status is the status of the condition. Can be True, False, Unknown.
                      type: string
                    type:
                      description: Type is the type of the condition.
                      type: string
                  type: object
                nullable: true
                type: array
              currentImage:
                type: string
              currentState:
                type: string
              evictionRequested:
                type: boolean
              instanceManagerName:
                type: string
              ip:
                type: string
              logFetched:
                type: boolean
              ownerID:
                type: string
              port:
                type: integer
              salvageExecuted:
                type: boolean
              started:
                type: boolean
              storageIP:
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: settings.longhorn.io
spec:
  group: longhorn.io
  names:
    kind: Setting
    listKind: SettingList
    plural: settings
    shortNames:
    - lhs
    singular: setting
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The value of the setting
      jsonPath: .value
      name: Value
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta1
    schema:
      openAPIV3Schema:
        description: Setting is where Longhorn stores setting object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          value:
            type: string
        required:
        - value
        type: object
    served: true
    storage: false
    subresources:
      status: {}
  - additionalPrinterColumns:
    - description: The value of the setting
      jsonPath: .value
      name: Value
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: Setting is where Longhorn stores setting object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          value:
            type: string
        required:
        - value
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: sharemanagers.longhorn.io
spec:
  group: longhorn.io
  names:
    kind: ShareManager
    listKind: ShareManagerList
    plural: sharemanagers
    shortNames:
    - lhsm
    singular: sharemanager
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The state of the share manager
      jsonPath: .status.state
      name: State
      type: string
    - description: The node that the share manager is owned by
      jsonPath: .status.ownerID
      name: Node
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta1
    schema:
      openAPIV3Schema:
        description: ShareManager is where Longhorn stores share manager object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            x-kubernetes-preserve-unknown-fields: true
          status:
            x-kubernetes-preserve-unknown-fields: true
        type: object
    served: true
    storage: false
    subresources:
      status: {}
  - additionalPrinterColumns:
    - description: The state of the share manager
      jsonPath: .status.state
      name: State
      type: string
    - description: The node that the share manager is owned by
      jsonPath: .status.ownerID
      name: Node
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: ShareManager is where Longhorn stores share manager object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: ShareManagerSpec defines the desired state of the Longhorn share manager
            properties:
              image:
                type: string
            type: object
          status:
            description: ShareManagerStatus defines the observed state of the Longhorn share manager
            properties:
              endpoint:
                type: string
              ownerID:
                type: string
              state:
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: snapshots.longhorn.io
spec:
  group: longhorn.io
  names:
    kind: Snapshot
    listKind: SnapshotList
    plural: snapshots
    shortNames:
    - lhsnap
    singular: snapshot
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The volume that this snapshot belongs to
      jsonPath: .spec.volume
      name: Volume
      type: string
    - description: Timestamp when the point-in-time snapshot was taken
      jsonPath: .status.creationTime
      name: CreationTime
      type: string
    - description: Indicates if the snapshot is ready to be used to restore/backup a volume
      jsonPath: .status.readyToUse
      name: ReadyToUse
      type: boolean
    - description: Represents the minimum size of volume required to rehydrate from this snapshot
      jsonPath: .status.restoreSize
      name: RestoreSize
      type: string
    - description: The actual size of the snapshot
      jsonPath: .status.size
      name: Size
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: Snapshot is the Schema for the snapshots API
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: SnapshotSpec defines the desired state of Longhorn Snapshot
            properties:
              createSnapshot:
                description: require creating a new snapshot
                type: boolean
              labels:
                additionalProperties:
                  type: string
                description: The labels of snapshot
                nullable: true
                type: object
              volume:
                description: the volume that this snapshot belongs to. This field is immutable after creation. Required
                type: string
            required:
            - volume
            type: object
          status:
            description: SnapshotStatus defines the observed state of Longhorn Snapshot
            properties:
              checksum:
                type: string
              children:
                additionalProperties:
                  type: boolean
                nullable: true
                type: object
              creationTime:
                type: string
              error:
                type: string
              labels:
                additionalProperties:
                  type: string
                nullable: true
                type: object
              markRemoved:
                type: boolean
              ownerID:
                type: string
              parent:
                type: string
              readyToUse:
                type: boolean
              restoreSize:
                format: int64
                type: integer
              size:
                format: int64
                type: integer
              userCreated:
                type: boolean
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: supportbundles.longhorn.io
spec:
  group: longhorn.io
  names:
    kind: SupportBundle
    listKind: SupportBundleList
    plural: supportbundles
    shortNames:
    - lhbundle
    singular: supportbundle
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The state of the support bundle
      jsonPath: .status.state
      name: State
      type: string
    - description: The issue URL
      jsonPath: .spec.issueURL
      name: Issue
      type: string
    - description: A brief description of the issue
      jsonPath: .spec.description
      name: Description
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: SupportBundle is where Longhorn stores support bundle object
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: SupportBundleSpec defines the desired state of the Longhorn SupportBundle
            properties:
              description:
                description: A brief description of the issue
                type: string
              issueURL:
                description: The issue URL
                nullable: true
                type: string
              nodeID:
                description: The preferred responsible controller node ID.
                type: string
            required:
            - description
            type: object
          status:
            description: SupportBundleStatus defines the observed state of the Longhorn SupportBundle
            properties:
              conditions:
                items:
                  properties:
                    lastProbeTime:
                      description: Last time we probed the condition.
                      type: string
                    lastTransitionTime:
                      description: Last time the condition transitioned from one status to another.
                      type: string
                    message:
                      description: Human-readable message indicating details about last transition.
                      type: string
                    reason:
                      description: Unique, one-word, CamelCase reason for the condition's last transition.
                      type: string
                    status:
                      description: Status is the status of the condition. Can be True, False, Unknown.
                      type: string
                    type:
                      description: Type is the type of the condition.
                      type: string
                  type: object
                type: array
              filename:
                type: string
              filesize:
                format: int64
                type: integer
              image:
                description: The support bundle manager image
                type: string
              managerIP:
                description: The support bundle manager IP
                type: string
              ownerID:
                description: The current responsible controller node ID
                type: string
              progress:
                type: integer
              state:
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: systembackups.longhorn.io
spec:
  group: longhorn.io
  names:
    kind: SystemBackup
    listKind: SystemBackupList
    plural: systembackups
    shortNames:
    - lhsb
    singular: systembackup
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The system backup Longhorn version
      jsonPath: .status.version
      name: Version
      type: string
    - description: The system backup state
      jsonPath: .status.state
      name: State
      type: string
    - description: The system backup creation time
      jsonPath: .status.createdAt
      name: Created
      type: string
    - description: The last time that the system backup was synced into the cluster
      jsonPath: .status.lastSyncedAt
      name: LastSyncedAt
      type: string
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: SystemBackup is where Longhorn stores system backup object
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: SystemBackupSpec defines the desired state of the Longhorn SystemBackup
            type: object
          status:
            description: SystemBackupStatus defines the observed state of the Longhorn SystemBackup
            properties:
              conditions:
                items:
                  properties:
                    lastProbeTime:
                      description: Last time we probed the condition.
                      type: string
                    lastTransitionTime:
                      description: Last time the condition transitioned from one status to another.
                      type: string
                    message:
                      description: Human-readable message indicating details about last transition.
                      type: string
                    reason:
                      description: Unique, one-word, CamelCase reason for the condition's last transition.
                      type: string
                    status:
                      description: Status is the status of the condition. Can be True, False, Unknown.
                      type: string
                    type:
                      description: Type is the type of the condition.
                      type: string
                  type: object
                nullable: true
                type: array
              createdAt:
                description: The system backup creation time.
                format: date-time
                type: string
              gitCommit:
                description: The saved Longhorn manager git commit.
                nullable: true
                type: string
              lastSyncedAt:
                description: The last time that the system backup was synced into the cluster.
                format: date-time
                nullable: true
                type: string
              managerImage:
                description: The saved manager image.
                type: string
              ownerID:
                description: The node ID of the responsible controller to reconcile this SystemBackup.
                type: string
              state:
                description: The system backup state.
                type: string
              version:
                description: The saved Longhorn version.
                nullable: true
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  creationTimestamp: null
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: systemrestores.longhorn.io
spec:
  group: longhorn.io
  names:
    kind: SystemRestore
    listKind: SystemRestoreList
    plural: systemrestores
    shortNames:
    - lhsr
    singular: systemrestore
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The system restore state
      jsonPath: .status.state
      name: State
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: SystemRestore is where Longhorn stores system restore object
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: SystemRestoreSpec defines the desired state of the Longhorn SystemRestore
            properties:
              systemBackup:
                description: The system backup name in the object store.
                type: string
            required:
            - systemBackup
            type: object
          status:
            description: SystemRestoreStatus defines the observed state of the Longhorn SystemRestore
            properties:
              conditions:
                items:
                  properties:
                    lastProbeTime:
                      description: Last time we probed the condition.
                      type: string
                    lastTransitionTime:
                      description: Last time the condition transitioned from one status to another.
                      type: string
                    message:
                      description: Human-readable message indicating details about last transition.
                      type: string
                    reason:
                      description: Unique, one-word, CamelCase reason for the condition's last transition.
                      type: string
                    status:
                      description: Status is the status of the condition. Can be True, False, Unknown.
                      type: string
                    type:
                      description: Type is the type of the condition.
                      type: string
                  type: object
                nullable: true
                type: array
              ownerID:
                description: The node ID of the responsible controller to reconcile this SystemRestore.
                type: string
              sourceURL:
                description: The source system backup URL.
                type: string
              state:
                description: The system restore state.
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.7.0
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    longhorn-manager: ""
  name: volumes.longhorn.io
spec:
  preserveUnknownFields: false
  conversion:
    strategy: Webhook
    webhook:
      clientConfig:
        service:
          name: longhorn-conversion-webhook
          namespace: {{ include "release_namespace" . }}
          path: /v1/webhook/conversion
          port: 9443
      conversionReviewVersions:
      - v1beta2
      - v1beta1
  group: longhorn.io
  names:
    kind: Volume
    listKind: VolumeList
    plural: volumes
    shortNames:
    - lhv
    singular: volume
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - description: The state of the volume
      jsonPath: .status.state
      name: State
      type: string
    - description: The robustness of the volume
      jsonPath: .status.robustness
      name: Robustness
      type: string
    - description: The scheduled condition of the volume
      jsonPath: .status.conditions['scheduled']['status']
      name: Scheduled
      type: string
    - description: The size of the volume
      jsonPath: .spec.size
      name: Size
      type: string
    - description: The node that the volume is currently attaching to
      jsonPath: .status.currentNodeID
      name: Node
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta1
    schema:
      openAPIV3Schema:
        description: Volume is where Longhorn stores volume object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            x-kubernetes-preserve-unknown-fields: true
          status:
            x-kubernetes-preserve-unknown-fields: true
        type: object
    served: true
    storage: false
    subresources:
      status: {}
  - additionalPrinterColumns:
    - description: The state of the volume
      jsonPath: .status.state
      name: State
      type: string
    - description: The robustness of the volume
      jsonPath: .status.robustness
      name: Robustness
      type: string
    - description: The scheduled condition of the volume
      jsonPath: .status.conditions[?(@.type=='Schedulable')].status
      name: Scheduled
      type: string
    - description: The size of the volume
      jsonPath: .spec.size
      name: Size
      type: string
    - description: The node that the volume is currently attaching to
      jsonPath: .status.currentNodeID
      name: Node
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1beta2
    schema:
      openAPIV3Schema:
        description: Volume is where Longhorn stores volume object.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: VolumeSpec defines the desired state of the Longhorn volume
            properties:
              Standby:
                type: boolean
              accessMode:
                enum:
                - rwo
                - rwx
                type: string
              backingImage:
                type: string
              baseImage:
                description: Deprecated. Rename to BackingImage
                type: string
              dataLocality:
                enum:
                - disabled
                - best-effort
                - strict-local
                type: string
              dataSource:
                type: string
              disableFrontend:
                type: boolean
              diskSelector:
                items:
                  type: string
                type: array
              encrypted:
                type: boolean
              engineImage:
                type: string
              fromBackup:
                type: string
              restoreVolumeRecurringJob:
                enum:
                - ignored
                - enabled
                - disabled
                type: string
              frontend:
                enum:
                - blockdev
                - iscsi
                - ""
                type: string
              lastAttachedBy:
                type: string
              migratable:
                type: boolean
              migrationNodeID:
                type: string
              nodeID:
                type: string
              nodeSelector:
                items:
                  type: string
                type: array
              numberOfReplicas:
                type: integer
              recurringJobs:
                description: Deprecated. Replaced by a separate resource named "RecurringJob"
                items:
                  description: 'VolumeRecurringJobSpec is a deprecated struct. TODO: Should be removed when recurringJobs gets removed from the volume       spec.'
                  properties:
                    concurrency:
                      type: integer
                    cron:
                      type: string
                    groups:
                      items:
                        type: string
                      type: array
                    labels:
                      additionalProperties:
                        type: string
                      type: object
                    name:
                      type: string
                    retain:
                      type: integer
                    task:
                      enum:
                      - snapshot
                      - backup
                      type: string
                  type: object
                type: array
              replicaAutoBalance:
                enum:
                - ignored
                - disabled
                - least-effort
                - best-effort
                type: string
              revisionCounterDisabled:
                type: boolean
              size:
                format: int64
                type: string
              snapshotDataIntegrity:
                enum:
                - ignored
                - disabled
                - enabled
                - fast-check
                type: string
              staleReplicaTimeout:
                type: integer
              unmapMarkSnapChainRemoved:
                enum:
                - ignored
                - disabled
                - enabled
                type: string
            type: object
          status:
            description: VolumeStatus defines the observed state of the Longhorn volume
            properties:
              actualSize:
                format: int64
                type: integer
              cloneStatus:
                properties:
                  snapshot:
                    type: string
                  sourceVolume:
                    type: string
                  state:
                    type: string
                type: object
              conditions:
                items:
                  properties:
                    lastProbeTime:
                      description: Last time we probed the condition.
                      type: string
                    lastTransitionTime:
                      description: Last time the condition transitioned from one status to another.
                      type: string
                    message:
                      description: Human-readable message indicating details about last transition.
                      type: string
                    reason:
                      description: Unique, one-word, CamelCase reason for the condition's last transition.
                      type: string
                    status:
                      description: Status is the status of the condition. Can be True, False, Unknown.
                      type: string
                    type:
                      description: Type is the type of the condition.
                      type: string
                  type: object
                nullable: true
                type: array
              currentImage:
                type: string
              currentNodeID:
                type: string
              expansionRequired:
                type: boolean
              frontendDisabled:
                type: boolean
              isStandby:
                type: boolean
              kubernetesStatus:
                properties:
                  lastPVCRefAt:
                    type: string
                  lastPodRefAt:
                    type: string
                  namespace:
                    description: determine if PVC/Namespace is history or not
                    type: string
                  pvName:
                    type: string
                  pvStatus:
                    type: string
                  pvcName:
                    type: string
                  workloadsStatus:
                    description: determine if Pod/Workload is history or not
                    items:
                      properties:
                        podName:
                          type: string
                        podStatus:
                          type: string
                        workloadName:
                          type: string
                        workloadType:
                          type: string
                      type: object
                    nullable: true
                    type: array
                type: object
              lastBackup:
                type: string
              lastBackupAt:
                type: string
              lastDegradedAt:
                type: string
              ownerID:
                type: string
              pendingNodeID:
                type: string
              remountRequestedAt:
                type: string
              restoreInitiated:
                type: boolean
              restoreRequired:
                type: boolean
              robustness:
                type: string
              shareEndpoint:
                type: string
              shareState:
                type: string
              state:
                type: string
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
</file>

<file path="charts/longhorn/templates/daemonset-sa.yaml">
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    app: longhorn-manager
  name: longhorn-manager
  namespace: {{ include "release_namespace" . }}
spec:
  selector:
    matchLabels:
      app: longhorn-manager
  template:
    metadata:
      labels: {{- include "longhorn.labels" . | nindent 8 }}
        app: longhorn-manager
      {{- with .Values.annotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
    spec:
      initContainers:
      - name: wait-longhorn-admission-webhook
        image: {{ template "registry_url" . }}{{ .Values.image.longhorn.manager.repository }}:{{ .Values.image.longhorn.manager.tag }}
        command: ['sh', '-c', 'while [ $(curl -m 1 -s -o /dev/null -w "%{http_code}" -k https://longhorn-admission-webhook:9443/v1/healthz) != "200" ]; do echo waiting; sleep 2; done']
      containers:
      - name: longhorn-manager
        image: {{ template "registry_url" . }}{{ .Values.image.longhorn.manager.repository }}:{{ .Values.image.longhorn.manager.tag }}
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        securityContext:
          privileged: true
        command:
        - longhorn-manager
        - -d
        {{- if eq .Values.longhornManager.log.format "json" }}
        - -j
        {{- end }}
        - daemon
        - --engine-image
        - "{{ template "registry_url" . }}{{ .Values.image.longhorn.engine.repository }}:{{ .Values.image.longhorn.engine.tag }}"
        - --instance-manager-image
        - "{{ template "registry_url" . }}{{ .Values.image.longhorn.instanceManager.repository }}:{{ .Values.image.longhorn.instanceManager.tag }}"
        - --share-manager-image
        - "{{ template "registry_url" . }}{{ .Values.image.longhorn.shareManager.repository }}:{{ .Values.image.longhorn.shareManager.tag }}"
        - --backing-image-manager-image
        - "{{ template "registry_url" . }}{{ .Values.image.longhorn.backingImageManager.repository }}:{{ .Values.image.longhorn.backingImageManager.tag }}"
        - --support-bundle-manager-image
        - "{{ template "registry_url" . }}{{ .Values.image.longhorn.supportBundleKit.repository }}:{{ .Values.image.longhorn.supportBundleKit.tag }}"
        - --manager-image
        - "{{ template "registry_url" . }}{{ .Values.image.longhorn.manager.repository }}:{{ .Values.image.longhorn.manager.tag }}"
        - --service-account
        - longhorn-service-account
        ports:
        - containerPort: 9500
          name: manager
        readinessProbe:
          tcpSocket:
            port: 9500
        volumeMounts:
        - name: dev
          mountPath: /host/dev/
        - name: proc
          mountPath: /host/proc/
        - name: longhorn
          mountPath: /var/lib/longhorn/
          mountPropagation: Bidirectional
        - name: longhorn-grpc-tls
          mountPath: /tls-files/
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
      volumes:
      - name: dev
        hostPath:
          path: /dev/
      - name: proc
        hostPath:
          path: /proc/
      - name: longhorn
        hostPath:
          path: /var/lib/longhorn/
      - name: longhorn-grpc-tls
        secret:
          secretName: longhorn-grpc-tls
          optional: true
      {{- if .Values.privateRegistry.registrySecret }}
      imagePullSecrets:
      - name: {{ .Values.privateRegistry.registrySecret }}
      {{- end }}
      {{- if .Values.longhornManager.priorityClass }}
      priorityClassName: {{ .Values.longhornManager.priorityClass | quote }}
      {{- end }}
      {{- if or .Values.longhornManager.tolerations .Values.global.cattle.windowsCluster.enabled }}
      tolerations:
        {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.tolerations }}
{{ toYaml .Values.global.cattle.windowsCluster.tolerations | indent 6 }}
        {{- end }}
        {{- if .Values.longhornManager.tolerations }}
{{ toYaml .Values.longhornManager.tolerations | indent 6 }}
        {{- end }}
      {{- end }}
      {{- if or .Values.longhornManager.nodeSelector .Values.global.cattle.windowsCluster.enabled }}
      nodeSelector:
        {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.nodeSelector }}
{{ toYaml .Values.global.cattle.windowsCluster.nodeSelector | indent 8 }}
        {{- end }}
        {{- if .Values.longhornManager.nodeSelector }}
{{ toYaml .Values.longhornManager.nodeSelector | indent 8 }}
        {{- end }}
      {{- end }}
      serviceAccountName: longhorn-service-account
  updateStrategy:
    rollingUpdate:
      maxUnavailable: "100%"
---
apiVersion: v1
kind: Service
metadata:
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    app: longhorn-manager
  name: longhorn-backend
  namespace: {{ include "release_namespace" . }}
  {{- if .Values.longhornManager.serviceAnnotations }}
  annotations:
{{ toYaml .Values.longhornManager.serviceAnnotations | indent 4 }}
  {{- end }}
spec:
  type: {{ .Values.service.manager.type }}
  sessionAffinity: ClientIP
  selector:
    app: longhorn-manager
  ports:
  - name: manager
    port: 9500
    targetPort: manager
    {{- if .Values.service.manager.nodePort }}
    nodePort: {{ .Values.service.manager.nodePort }}
    {{- end }}
</file>

<file path="charts/longhorn/templates/default-setting.yaml">
apiVersion: v1
kind: ConfigMap
metadata:
  name: longhorn-default-setting
  namespace: {{ include "release_namespace" . }}
  labels: {{- include "longhorn.labels" . | nindent 4 }}
data:
  default-setting.yaml: |-
    {{ if not (kindIs "invalid" .Values.defaultSettings.backupTarget) }}backup-target: {{ .Values.defaultSettings.backupTarget }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.backupTargetCredentialSecret) }}backup-target-credential-secret: {{ .Values.defaultSettings.backupTargetCredentialSecret }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.allowRecurringJobWhileVolumeDetached) }}allow-recurring-job-while-volume-detached: {{ .Values.defaultSettings.allowRecurringJobWhileVolumeDetached }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.createDefaultDiskLabeledNodes) }}create-default-disk-labeled-nodes: {{ .Values.defaultSettings.createDefaultDiskLabeledNodes }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.defaultDataPath) }}default-data-path: {{ .Values.defaultSettings.defaultDataPath }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.replicaSoftAntiAffinity) }}replica-soft-anti-affinity: {{ .Values.defaultSettings.replicaSoftAntiAffinity }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.replicaAutoBalance) }}replica-auto-balance: {{ .Values.defaultSettings.replicaAutoBalance }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.storageOverProvisioningPercentage) }}storage-over-provisioning-percentage: {{ .Values.defaultSettings.storageOverProvisioningPercentage }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.storageMinimalAvailablePercentage) }}storage-minimal-available-percentage: {{ .Values.defaultSettings.storageMinimalAvailablePercentage }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.upgradeChecker) }}upgrade-checker: {{ .Values.defaultSettings.upgradeChecker }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.defaultReplicaCount) }}default-replica-count: {{ .Values.defaultSettings.defaultReplicaCount }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.defaultDataLocality) }}default-data-locality: {{ .Values.defaultSettings.defaultDataLocality }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.defaultLonghornStaticStorageClass) }}default-longhorn-static-storage-class: {{ .Values.defaultSettings.defaultLonghornStaticStorageClass }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.backupstorePollInterval) }}backupstore-poll-interval: {{ .Values.defaultSettings.backupstorePollInterval }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.failedBackupTTL) }}failed-backup-ttl: {{ .Values.defaultSettings.failedBackupTTL }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.restoreVolumeRecurringJobs) }}restore-volume-recurring-jobs: {{ .Values.defaultSettings.restoreVolumeRecurringJobs }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.recurringSuccessfulJobsHistoryLimit) }}recurring-successful-jobs-history-limit: {{ .Values.defaultSettings.recurringSuccessfulJobsHistoryLimit }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.recurringFailedJobsHistoryLimit) }}recurring-failed-jobs-history-limit: {{ .Values.defaultSettings.recurringFailedJobsHistoryLimit }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.supportBundleFailedHistoryLimit) }}support-bundle-failed-history-limit: {{ .Values.defaultSettings.supportBundleFailedHistoryLimit }}{{ end }}
    {{- if or (not (kindIs "invalid" .Values.defaultSettings.taintToleration)) (.Values.global.cattle.windowsCluster.enabled) }}
    taint-toleration: {{ $windowsDefaultSettingTaintToleration := list }}{{ $defaultSettingTaintToleration := list -}}
      {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.defaultSetting.taintToleration -}}
        {{- $windowsDefaultSettingTaintToleration = .Values.global.cattle.windowsCluster.defaultSetting.taintToleration -}}
      {{- end -}}
      {{- if not (kindIs "invalid" .Values.defaultSettings.taintToleration) -}}
        {{- $defaultSettingTaintToleration = .Values.defaultSettings.taintToleration -}}
      {{- end -}}
      {{- $taintToleration := list $windowsDefaultSettingTaintToleration $defaultSettingTaintToleration }}{{ join ";" (compact $taintToleration) -}}
    {{- end }}
    {{- if or (not (kindIs "invalid" .Values.defaultSettings.systemManagedComponentsNodeSelector)) (.Values.global.cattle.windowsCluster.enabled) }}
    system-managed-components-node-selector: {{ $windowsDefaultSettingNodeSelector := list }}{{ $defaultSettingNodeSelector := list -}}
      {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.defaultSetting.systemManagedComponentsNodeSelector -}}
        {{ $windowsDefaultSettingNodeSelector = .Values.global.cattle.windowsCluster.defaultSetting.systemManagedComponentsNodeSelector -}}
      {{- end -}}
      {{- if not (kindIs "invalid" .Values.defaultSettings.systemManagedComponentsNodeSelector) -}}
        {{- $defaultSettingNodeSelector = .Values.defaultSettings.systemManagedComponentsNodeSelector -}}
      {{- end -}}
      {{- $nodeSelector := list $windowsDefaultSettingNodeSelector $defaultSettingNodeSelector }}{{ join ";" (compact $nodeSelector) -}}
    {{- end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.priorityClass) }}priority-class: {{ .Values.defaultSettings.priorityClass }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.autoSalvage) }}auto-salvage: {{ .Values.defaultSettings.autoSalvage }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.autoDeletePodWhenVolumeDetachedUnexpectedly) }}auto-delete-pod-when-volume-detached-unexpectedly: {{ .Values.defaultSettings.autoDeletePodWhenVolumeDetachedUnexpectedly }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.disableSchedulingOnCordonedNode) }}disable-scheduling-on-cordoned-node: {{ .Values.defaultSettings.disableSchedulingOnCordonedNode }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.replicaZoneSoftAntiAffinity) }}replica-zone-soft-anti-affinity: {{ .Values.defaultSettings.replicaZoneSoftAntiAffinity }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.nodeDownPodDeletionPolicy) }}node-down-pod-deletion-policy: {{ .Values.defaultSettings.nodeDownPodDeletionPolicy }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.allowNodeDrainWithLastHealthyReplica) }}allow-node-drain-with-last-healthy-replica: {{ .Values.defaultSettings.allowNodeDrainWithLastHealthyReplica }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.mkfsExt4Parameters) }}mkfs-ext4-parameters: {{ .Values.defaultSettings.mkfsExt4Parameters }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.disableReplicaRebuild) }}disable-replica-rebuild: {{ .Values.defaultSettings.disableReplicaRebuild }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.replicaReplenishmentWaitInterval) }}replica-replenishment-wait-interval: {{ .Values.defaultSettings.replicaReplenishmentWaitInterval }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.concurrentReplicaRebuildPerNodeLimit) }}concurrent-replica-rebuild-per-node-limit: {{ .Values.defaultSettings.concurrentReplicaRebuildPerNodeLimit }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.concurrentVolumeBackupRestorePerNodeLimit) }}concurrent-volume-backup-restore-per-node-limit: {{ .Values.defaultSettings.concurrentVolumeBackupRestorePerNodeLimit }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.disableRevisionCounter) }}disable-revision-counter: {{ .Values.defaultSettings.disableRevisionCounter }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.systemManagedPodsImagePullPolicy) }}system-managed-pods-image-pull-policy: {{ .Values.defaultSettings.systemManagedPodsImagePullPolicy }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.allowVolumeCreationWithDegradedAvailability) }}allow-volume-creation-with-degraded-availability: {{ .Values.defaultSettings.allowVolumeCreationWithDegradedAvailability }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.autoCleanupSystemGeneratedSnapshot) }}auto-cleanup-system-generated-snapshot: {{ .Values.defaultSettings.autoCleanupSystemGeneratedSnapshot }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.concurrentAutomaticEngineUpgradePerNodeLimit) }}concurrent-automatic-engine-upgrade-per-node-limit: {{ .Values.defaultSettings.concurrentAutomaticEngineUpgradePerNodeLimit }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.backingImageCleanupWaitInterval) }}backing-image-cleanup-wait-interval: {{ .Values.defaultSettings.backingImageCleanupWaitInterval }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.backingImageRecoveryWaitInterval) }}backing-image-recovery-wait-interval: {{ .Values.defaultSettings.backingImageRecoveryWaitInterval }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.guaranteedEngineManagerCPU) }}guaranteed-engine-manager-cpu: {{ .Values.defaultSettings.guaranteedEngineManagerCPU }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.guaranteedReplicaManagerCPU) }}guaranteed-replica-manager-cpu: {{ .Values.defaultSettings.guaranteedReplicaManagerCPU }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.kubernetesClusterAutoscalerEnabled) }}kubernetes-cluster-autoscaler-enabled: {{ .Values.defaultSettings.kubernetesClusterAutoscalerEnabled }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.orphanAutoDeletion) }}orphan-auto-deletion: {{ .Values.defaultSettings.orphanAutoDeletion }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.storageNetwork) }}storage-network: {{ .Values.defaultSettings.storageNetwork }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.deletingConfirmationFlag) }}deleting-confirmation-flag: {{ .Values.defaultSettings.deletingConfirmationFlag }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.engineReplicaTimeout) }}engine-replica-timeout: {{ .Values.defaultSettings.engineReplicaTimeout }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.snapshotDataIntegrity) }}snapshot-data-integrity: {{ .Values.defaultSettings.snapshotDataIntegrity }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.snapshotDataIntegrityImmediateCheckAfterSnapshotCreation) }}snapshot-data-integrity-immediate-check-after-snapshot-creation: {{ .Values.defaultSettings.snapshotDataIntegrityImmediateCheckAfterSnapshotCreation }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.snapshotDataIntegrityCronjob) }}snapshot-data-integrity-cronjob: {{ .Values.defaultSettings.snapshotDataIntegrityCronjob }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.removeSnapshotsDuringFilesystemTrim) }}remove-snapshots-during-filesystem-trim: {{ .Values.defaultSettings.removeSnapshotsDuringFilesystemTrim }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.fastReplicaRebuildEnabled) }}fast-replica-rebuild-enabled: {{ .Values.defaultSettings.fastReplicaRebuildEnabled }}{{ end }}
    {{ if not (kindIs "invalid" .Values.defaultSettings.replicaFileSyncHttpClientTimeout) }}replica-file-sync-http-client-timeout: {{ .Values.defaultSettings.replicaFileSyncHttpClientTimeout }}{{ end }}
</file>

<file path="charts/longhorn/templates/deployment-driver.yaml">
apiVersion: apps/v1
kind: Deployment
metadata:
  name: longhorn-driver-deployer
  namespace: {{ include "release_namespace" . }}
  labels: {{- include "longhorn.labels" . | nindent 4 }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: longhorn-driver-deployer
  template:
    metadata:
      labels: {{- include "longhorn.labels" . | nindent 8 }}
        app: longhorn-driver-deployer
    spec:
      initContainers:
        - name: wait-longhorn-manager
          image: {{ template "registry_url" . }}{{ .Values.image.longhorn.manager.repository }}:{{ .Values.image.longhorn.manager.tag }}
          command: ['sh', '-c', 'while [ $(curl -m 1 -s -o /dev/null -w "%{http_code}" http://longhorn-backend:9500/v1) != "200" ]; do echo waiting; sleep 2; done']
      containers:
        - name: longhorn-driver-deployer
          image: {{ template "registry_url" . }}{{ .Values.image.longhorn.manager.repository }}:{{ .Values.image.longhorn.manager.tag }}
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          command:
          - longhorn-manager
          - -d
          - deploy-driver
          - --manager-image
          - "{{ template "registry_url" . }}{{ .Values.image.longhorn.manager.repository }}:{{ .Values.image.longhorn.manager.tag }}"
          - --manager-url
          - http://longhorn-backend:9500/v1
          env:
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          - name: NODE_NAME
            valueFrom:
              fieldRef:
                fieldPath: spec.nodeName
          - name: SERVICE_ACCOUNT
            valueFrom:
              fieldRef:
                fieldPath: spec.serviceAccountName
          {{- if .Values.csi.kubeletRootDir }}
          - name: KUBELET_ROOT_DIR
            value: {{ .Values.csi.kubeletRootDir }}
          {{- end }}
          {{- if and .Values.image.csi.attacher.repository .Values.image.csi.attacher.tag }}
          - name: CSI_ATTACHER_IMAGE
            value: "{{ template "registry_url" . }}{{ .Values.image.csi.attacher.repository }}:{{ .Values.image.csi.attacher.tag }}"
          {{- end }}
          {{- if and .Values.image.csi.provisioner.repository .Values.image.csi.provisioner.tag }}
          - name: CSI_PROVISIONER_IMAGE
            value: "{{ template "registry_url" . }}{{ .Values.image.csi.provisioner.repository }}:{{ .Values.image.csi.provisioner.tag }}"
          {{- end }}
          {{- if and .Values.image.csi.nodeDriverRegistrar.repository .Values.image.csi.nodeDriverRegistrar.tag }}
          - name: CSI_NODE_DRIVER_REGISTRAR_IMAGE
            value: "{{ template "registry_url" . }}{{ .Values.image.csi.nodeDriverRegistrar.repository }}:{{ .Values.image.csi.nodeDriverRegistrar.tag }}"
          {{- end }}
          {{- if and .Values.image.csi.resizer.repository .Values.image.csi.resizer.tag }}
          - name: CSI_RESIZER_IMAGE
            value: "{{ template "registry_url" . }}{{ .Values.image.csi.resizer.repository }}:{{ .Values.image.csi.resizer.tag }}"
          {{- end }}
          {{- if and .Values.image.csi.snapshotter.repository .Values.image.csi.snapshotter.tag }}
          - name: CSI_SNAPSHOTTER_IMAGE
            value: "{{ template "registry_url" . }}{{ .Values.image.csi.snapshotter.repository }}:{{ .Values.image.csi.snapshotter.tag }}"
          {{- end }}
          {{- if and .Values.image.csi.livenessProbe.repository .Values.image.csi.livenessProbe.tag }}
          - name: CSI_LIVENESS_PROBE_IMAGE
            value: "{{ template "registry_url" . }}{{ .Values.image.csi.livenessProbe.repository }}:{{ .Values.image.csi.livenessProbe.tag }}"
          {{- end }}
          {{- if .Values.csi.attacherReplicaCount }}
          - name: CSI_ATTACHER_REPLICA_COUNT
            value: {{ .Values.csi.attacherReplicaCount | quote }}
          {{- end }}
          {{- if .Values.csi.provisionerReplicaCount }}
          - name: CSI_PROVISIONER_REPLICA_COUNT
            value: {{ .Values.csi.provisionerReplicaCount | quote }}
          {{- end }}
          {{- if .Values.csi.resizerReplicaCount }}
          - name: CSI_RESIZER_REPLICA_COUNT
            value: {{ .Values.csi.resizerReplicaCount | quote }}
          {{- end }}
          {{- if .Values.csi.snapshotterReplicaCount }}
          - name: CSI_SNAPSHOTTER_REPLICA_COUNT
            value: {{ .Values.csi.snapshotterReplicaCount | quote }}
          {{- end }}

      {{- if .Values.privateRegistry.registrySecret }}
      imagePullSecrets:
      - name: {{ .Values.privateRegistry.registrySecret }}
      {{- end }}
      {{- if .Values.longhornDriver.priorityClass }}
      priorityClassName: {{ .Values.longhornDriver.priorityClass | quote }}
      {{- end }}
      {{- if or .Values.longhornDriver.tolerations .Values.global.cattle.windowsCluster.enabled }}
      tolerations:
        {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.tolerations }}
{{ toYaml .Values.global.cattle.windowsCluster.tolerations | indent 6 }}
        {{- end }}
        {{- if .Values.longhornDriver.tolerations }}
{{ toYaml .Values.longhornDriver.tolerations | indent 6 }}
        {{- end }}
      {{- end }}
      {{- if or .Values.longhornDriver.nodeSelector .Values.global.cattle.windowsCluster.enabled }}
      nodeSelector:
        {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.nodeSelector }}
{{ toYaml .Values.global.cattle.windowsCluster.nodeSelector | indent 8 }}
        {{- end }}
        {{- if .Values.longhornDriver.nodeSelector }}
{{ toYaml .Values.longhornDriver.nodeSelector | indent 8 }}
        {{- end }}
      {{- end }}
      serviceAccountName: longhorn-service-account
      securityContext:
        runAsUser: 0
</file>

<file path="charts/longhorn/templates/deployment-recovery-backend.yaml">
apiVersion: apps/v1
kind: Deployment
metadata:
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    app: longhorn-recovery-backend
  name: longhorn-recovery-backend
  namespace: {{ include "release_namespace" . }}
spec:
  replicas: {{ .Values.longhornRecoveryBackend.replicas }}
  selector:
    matchLabels:
      app: longhorn-recovery-backend
  template:
    metadata:
      labels: {{- include "longhorn.labels" . | nindent 8 }}
        app: longhorn-recovery-backend
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - longhorn-recovery-backend
              topologyKey: kubernetes.io/hostname
      containers:
      - name: longhorn-recovery-backend
        image: {{ template "registry_url" . }}{{ .Values.image.longhorn.manager.repository }}:{{ .Values.image.longhorn.manager.tag }}
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        securityContext:
          runAsUser: 2000
        command:
        - longhorn-manager
        - recovery-backend
        - --service-account
        - longhorn-service-account
        ports:
        - containerPort: 9600
          name: recov-backend
        readinessProbe:
          tcpSocket:
            port: 9600
          initialDelaySeconds: 3
          periodSeconds: 5
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
      {{- if .Values.privateRegistry.registrySecret }}
      imagePullSecrets:
      - name: {{ .Values.privateRegistry.registrySecret }}
      {{- end }}
      {{- if .Values.longhornRecoveryBackend.priorityClass }}
      priorityClassName: {{ .Values.longhornRecoveryBackend.priorityClass | quote }}
      {{- end }}
      {{- if or .Values.longhornRecoveryBackend.tolerations .Values.global.cattle.windowsCluster.enabled }}
      tolerations:
        {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.tolerations }}
{{ toYaml .Values.global.cattle.windowsCluster.tolerations | indent 6 }}
        {{- end }}
        {{- if .Values.longhornRecoveryBackend.tolerations }}
{{ toYaml .Values.longhornRecoveryBackend.tolerations | indent 6 }}
        {{- end }}
      {{- end }}
      {{- if or .Values.longhornRecoveryBackend.nodeSelector .Values.global.cattle.windowsCluster.enabled }}
      nodeSelector:
        {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.nodeSelector }}
{{ toYaml .Values.global.cattle.windowsCluster.nodeSelector | indent 8 }}
        {{- end }}
        {{- if .Values.longhornRecoveryBackend.nodeSelector }}
{{ toYaml .Values.longhornRecoveryBackend.nodeSelector | indent 8 }}
        {{- end }}
      {{- end }}
      serviceAccountName: longhorn-service-account
</file>

<file path="charts/longhorn/templates/deployment-ui.yaml">
apiVersion: apps/v1
kind: Deployment
metadata:
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    app: longhorn-ui
  name: longhorn-ui
  namespace: {{ include "release_namespace" . }}
spec:
  replicas: {{ .Values.longhornUI.replicas }}
  selector:
    matchLabels:
      app: longhorn-ui
  template:
    metadata:
      labels: {{- include "longhorn.labels" . | nindent 8 }}
        app: longhorn-ui
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - longhorn-ui
              topologyKey: kubernetes.io/hostname
      containers:
      - name: longhorn-ui
        image: {{ template "registry_url" . }}{{ .Values.image.longhorn.ui.repository }}:{{ .Values.image.longhorn.ui.tag }}
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        volumeMounts:
        - name : nginx-cache
          mountPath: /var/cache/nginx/
        - name : nginx-config
          mountPath: /var/config/nginx/
        - name: var-run
          mountPath: /var/run/
        ports:
        - containerPort: 8000
          name: http
        env:
          - name: LONGHORN_MANAGER_IP
            value: "http://longhorn-backend:9500"
          - name: LONGHORN_UI_PORT
            value: "8000"
      volumes:
      - emptyDir: {}
        name: nginx-cache
      - emptyDir: {}
        name: nginx-config
      - emptyDir: {}
        name: var-run
      {{- if .Values.privateRegistry.registrySecret }}
      imagePullSecrets:
      - name: {{ .Values.privateRegistry.registrySecret }}
      {{- end }}
      {{- if .Values.longhornUI.priorityClass }}
      priorityClassName: {{ .Values.longhornUI.priorityClass | quote }}
      {{- end }}
      {{- if or .Values.longhornUI.tolerations .Values.global.cattle.windowsCluster.enabled }}
      tolerations:
        {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.tolerations }}
{{ toYaml .Values.global.cattle.windowsCluster.tolerations | indent 6 }}
        {{- end }}
        {{- if .Values.longhornUI.tolerations }}
{{ toYaml .Values.longhornUI.tolerations | indent 6 }}
        {{- end }}
      {{- end }}
      {{- if or .Values.longhornUI.nodeSelector .Values.global.cattle.windowsCluster.enabled }}
      nodeSelector:
        {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.nodeSelector }}
{{ toYaml .Values.global.cattle.windowsCluster.nodeSelector | indent 8 }}
        {{- end }}
        {{- if .Values.longhornUI.nodeSelector }}
{{ toYaml .Values.longhornUI.nodeSelector | indent 8 }}
        {{- end }}
      {{- end }}
---
kind: Service
apiVersion: v1
metadata:
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    app: longhorn-ui
    {{- if eq .Values.service.ui.type "Rancher-Proxy" }}
    kubernetes.io/cluster-service: "true"
    {{- end }}
  name: longhorn-frontend
  namespace: {{ include "release_namespace" . }}
spec:
  {{- if eq .Values.service.ui.type "Rancher-Proxy" }}
  type: ClusterIP
  {{- else }}
  type: {{ .Values.service.ui.type }}
  {{- end }}
  {{- if and .Values.service.ui.loadBalancerIP (eq .Values.service.ui.type "LoadBalancer") }}
  loadBalancerIP: {{ .Values.service.ui.loadBalancerIP }}
  {{- end }}
  {{- if and (eq .Values.service.ui.type "LoadBalancer") .Values.service.ui.loadBalancerSourceRanges }}
  loadBalancerSourceRanges: {{- toYaml .Values.service.ui.loadBalancerSourceRanges | nindent 4 }}
  {{- end }}
  selector:
    app: longhorn-ui
  ports:
  - name: http
    port: 80
    targetPort: http
    {{- if .Values.service.ui.nodePort }}
    nodePort: {{ .Values.service.ui.nodePort }}
    {{- else }}
    nodePort: null
    {{- end }}
</file>

<file path="charts/longhorn/templates/deployment-webhook.yaml">
apiVersion: apps/v1
kind: Deployment
metadata:
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    app: longhorn-conversion-webhook
  name: longhorn-conversion-webhook
  namespace: {{ include "release_namespace" . }}
spec:
  replicas: {{ .Values.longhornConversionWebhook.replicas }}
  selector:
    matchLabels:
      app: longhorn-conversion-webhook
  template:
    metadata:
      labels: {{- include "longhorn.labels" . | nindent 8 }}
        app: longhorn-conversion-webhook
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - longhorn-conversion-webhook
              topologyKey: kubernetes.io/hostname
      containers:
      - name: longhorn-conversion-webhook
        image: {{ template "registry_url" . }}{{ .Values.image.longhorn.manager.repository }}:{{ .Values.image.longhorn.manager.tag }}
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        securityContext:
          runAsUser: 2000
        command:
        - longhorn-manager
        - conversion-webhook
        - --service-account
        - longhorn-service-account
        ports:
        - containerPort: 9443
          name: conversion-wh
        readinessProbe:
          tcpSocket:
            port: 9443
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
      {{- if .Values.privateRegistry.registrySecret }}
      imagePullSecrets:
      - name: {{ .Values.privateRegistry.registrySecret }}
      {{- end }}
      {{- if .Values.longhornConversionWebhook.priorityClass }}
      priorityClassName: {{ .Values.longhornConversionWebhook.priorityClass | quote }}
      {{- end }}
      {{- if or .Values.longhornConversionWebhook.tolerations .Values.global.cattle.windowsCluster.enabled }}
      tolerations:
        {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.tolerations }}
{{ toYaml .Values.global.cattle.windowsCluster.tolerations | indent 6 }}
        {{- end }}
        {{- if .Values.longhornConversionWebhook.tolerations }}
{{ toYaml .Values.longhornConversionWebhook.tolerations | indent 6 }}
        {{- end }}
      {{- end }}
      {{- if or .Values.longhornConversionWebhook.nodeSelector .Values.global.cattle.windowsCluster.enabled }}
      nodeSelector:
        {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.nodeSelector }}
{{ toYaml .Values.global.cattle.windowsCluster.nodeSelector | indent 8 }}
        {{- end }}
        {{- if .Values.longhornConversionWebhook.nodeSelector }}
{{ toYaml .Values.longhornConversionWebhook.nodeSelector | indent 8 }}
        {{- end }}
      {{- end }}
      serviceAccountName: longhorn-service-account
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    app: longhorn-admission-webhook
  name: longhorn-admission-webhook
  namespace: {{ include "release_namespace" . }}
spec:
  replicas: {{ .Values.longhornAdmissionWebhook.replicas }}
  selector:
    matchLabels:
      app: longhorn-admission-webhook
  template:
    metadata:
      labels: {{- include "longhorn.labels" . | nindent 8 }}
        app: longhorn-admission-webhook
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - longhorn-admission-webhook
              topologyKey: kubernetes.io/hostname
      initContainers:
      - name: wait-longhorn-conversion-webhook
        image: {{ template "registry_url" . }}{{ .Values.image.longhorn.manager.repository }}:{{ .Values.image.longhorn.manager.tag }}
        command: ['sh', '-c', 'while [ $(curl -m 1 -s -o /dev/null -w "%{http_code}" -k https://longhorn-conversion-webhook:9443/v1/healthz) != "200" ]; do echo waiting; sleep 2; done']
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        securityContext:
          runAsUser: 2000
      containers:
      - name: longhorn-admission-webhook
        image: {{ template "registry_url" . }}{{ .Values.image.longhorn.manager.repository }}:{{ .Values.image.longhorn.manager.tag }}
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        securityContext:
          runAsUser: 2000
        command:
        - longhorn-manager
        - admission-webhook
        - --service-account
        - longhorn-service-account
        ports:
        - containerPort: 9443
          name: admission-wh
        readinessProbe:
          tcpSocket:
            port: 9443
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
      {{- if .Values.privateRegistry.registrySecret }}
      imagePullSecrets:
      - name: {{ .Values.privateRegistry.registrySecret }}
      {{- end }}
      {{- if .Values.longhornAdmissionWebhook.priorityClass }}
      priorityClassName: {{ .Values.longhornAdmissionWebhook.priorityClass | quote }}
      {{- end }}
      {{- if or .Values.longhornAdmissionWebhook.tolerations .Values.global.cattle.windowsCluster.enabled }}
      tolerations:
        {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.tolerations }}
{{ toYaml .Values.global.cattle.windowsCluster.tolerations | indent 6 }}
        {{- end }}
        {{- if .Values.longhornAdmissionWebhook.tolerations }}
{{ toYaml .Values.longhornAdmissionWebhook.tolerations | indent 6 }}
        {{- end }}
      {{- end }}
      {{- if or .Values.longhornAdmissionWebhook.nodeSelector .Values.global.cattle.windowsCluster.enabled }}
      nodeSelector:
        {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.nodeSelector }}
{{ toYaml .Values.global.cattle.windowsCluster.nodeSelector | indent 8 }}
        {{- end }}
        {{- if .Values.longhornAdmissionWebhook.nodeSelector }}
{{ toYaml .Values.longhornAdmissionWebhook.nodeSelector | indent 8 }}
        {{- end }}
      {{- end }}
      serviceAccountName: longhorn-service-account
</file>

<file path="charts/longhorn/templates/ingress.yaml">
{{- if .Values.ingress.enabled }}
{{- if semverCompare ">=1.19-0" .Capabilities.KubeVersion.GitVersion -}}
apiVersion: networking.k8s.io/v1
{{- else -}}
apiVersion: networking.k8s.io/v1beta1
{{- end }}
kind: Ingress
metadata:
  name: longhorn-ingress
  namespace: {{ include "release_namespace" . }}
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    app: longhorn-ingress
  annotations:
    {{- if .Values.ingress.secureBackends }}
    ingress.kubernetes.io/secure-backends: "true"
    {{- end }}
    {{- range $key, $value := .Values.ingress.annotations }}
    {{ $key }}: {{ $value | quote }}
    {{- end }}
spec:
  {{- if and .Values.ingress.ingressClassName (semverCompare ">=1.18-0" .Capabilities.KubeVersion.GitVersion) }}
  ingressClassName: {{ .Values.ingress.ingressClassName }}
  {{- end }}
  rules:
  - host: {{ .Values.ingress.host }}
    http:
      paths:
        - path: {{ default "" .Values.ingress.path }}
          {{- if (semverCompare ">=1.18-0" $.Capabilities.KubeVersion.GitVersion) }}
          pathType: ImplementationSpecific
          {{- end }}
          backend:
            {{- if semverCompare ">=1.19-0" $.Capabilities.KubeVersion.GitVersion }}
            service:
              name: longhorn-frontend
              port:
                number: 80
            {{- else }}
            serviceName: longhorn-frontend
            servicePort: 80
            {{- end }}
{{- if .Values.ingress.tls }}
  tls:
  - hosts:
    - {{ .Values.ingress.host }}
    secretName: {{ .Values.ingress.tlsSecret }}
{{- end }}
{{- end }}
</file>

<file path="charts/longhorn/templates/NOTES.txt">
Longhorn is now installed on the cluster!

Please wait a few minutes for other Longhorn components such as CSI deployments, Engine Images, and Instance Managers to be initialized.

Visit our documentation at https://longhorn.io/docs/
</file>

<file path="charts/longhorn/templates/postupgrade-job.yaml">
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    "helm.sh/hook": post-upgrade
    "helm.sh/hook-delete-policy": hook-succeeded,before-hook-creation
  name: longhorn-post-upgrade
  namespace: {{ include "release_namespace" . }}
  labels: {{- include "longhorn.labels" . | nindent 4 }}
spec:
  activeDeadlineSeconds: 900
  backoffLimit: 1
  template:
    metadata:
      name: longhorn-post-upgrade
      labels: {{- include "longhorn.labels" . | nindent 8 }}
    spec:
      containers:
      - name: longhorn-post-upgrade
        image: {{ template "registry_url" . }}{{ .Values.image.longhorn.manager.repository }}:{{ .Values.image.longhorn.manager.tag }}
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        securityContext:
          privileged: true
        command:
        - longhorn-manager
        - post-upgrade
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
      restartPolicy: OnFailure
      {{- if .Values.privateRegistry.registrySecret }}
      imagePullSecrets:
      - name: {{ .Values.privateRegistry.registrySecret }}
      {{- end }}
      {{- if .Values.longhornManager.priorityClass }}
      priorityClassName: {{ .Values.longhornManager.priorityClass | quote }}
      {{- end }}
      serviceAccountName: longhorn-service-account
      {{- if or .Values.longhornManager.tolerations .Values.global.cattle.windowsCluster.enabled }}
      tolerations:
        {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.tolerations }}
{{ toYaml .Values.global.cattle.windowsCluster.tolerations | indent 6 }}
        {{- end }}
        {{- if .Values.longhornManager.tolerations }}
{{ toYaml .Values.longhornManager.tolerations | indent 6 }}
        {{- end }}
      {{- end }}
      {{- if or .Values.longhornManager.nodeSelector .Values.global.cattle.windowsCluster.enabled }}
      nodeSelector:
        {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.nodeSelector }}
{{ toYaml .Values.global.cattle.windowsCluster.nodeSelector | indent 8 }}
        {{- end }}
        {{- if .Values.longhornManager.nodeSelector }}
{{ toYaml .Values.longhornManager.nodeSelector | indent 8 }}
        {{- end }}
      {{- end }}
</file>

<file path="charts/longhorn/templates/psp.yaml">
{{- if .Values.enablePSP }}
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: longhorn-psp
  labels: {{- include "longhorn.labels" . | nindent 4 }}
spec:
  privileged: true
  allowPrivilegeEscalation: true
  requiredDropCapabilities:
  - NET_RAW
  allowedCapabilities:
  - SYS_ADMIN
  hostNetwork: false
  hostIPC: false
  hostPID: true
  runAsUser:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  volumes:
  - configMap
  - downwardAPI
  - emptyDir
  - secret
  - projected
  - hostPath
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: longhorn-psp-role
  labels: {{- include "longhorn.labels" . | nindent 4 }}
  namespace: {{ include "release_namespace" . }}
rules:
- apiGroups:
  - policy
  resources:
  - podsecuritypolicies
  verbs:
  - use
  resourceNames:
  - longhorn-psp
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: longhorn-psp-binding
  labels: {{- include "longhorn.labels" . | nindent 4 }}
  namespace: {{ include "release_namespace" . }}
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: longhorn-psp-role
subjects:
- kind: ServiceAccount
  name: longhorn-service-account
  namespace: {{ include "release_namespace" . }}
- kind: ServiceAccount
  name: default
  namespace: {{ include "release_namespace" . }}
{{- end }}
</file>

<file path="charts/longhorn/templates/registry-secret.yaml">
{{- if .Values.privateRegistry.createSecret }}
{{- if .Values.privateRegistry.registrySecret }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Values.privateRegistry.registrySecret }}
  namespace: {{ include "release_namespace" . }}
  labels: {{- include "longhorn.labels" . | nindent 4 }}
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: {{ template "secret" . }}
{{- end }}
{{- end }}
</file>

<file path="charts/longhorn/templates/serviceaccount.yaml">
apiVersion: v1
kind: ServiceAccount
metadata:
  name: longhorn-service-account
  namespace: {{ include "release_namespace" . }}
  labels: {{- include "longhorn.labels" . | nindent 4 }}
  {{- with .Values.serviceAccount.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: longhorn-support-bundle
  namespace: {{ include "release_namespace" . }}
  labels: {{- include "longhorn.labels" . | nindent 4 }}
  {{- with .Values.serviceAccount.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
</file>

<file path="charts/longhorn/templates/services.yaml">
apiVersion: v1
kind: Service
metadata:
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    app: longhorn-conversion-webhook
  name: longhorn-conversion-webhook
  namespace: {{ include "release_namespace" . }}
spec:
  type: ClusterIP
  sessionAffinity: ClientIP
  selector:
    app: longhorn-conversion-webhook
  ports:
  - name: conversion-webhook
    port: 9443
    targetPort: conversion-wh
---
apiVersion: v1
kind: Service
metadata:
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    app: longhorn-admission-webhook
  name: longhorn-admission-webhook
  namespace: {{ include "release_namespace" . }}
spec:
  type: ClusterIP
  sessionAffinity: ClientIP
  selector:
    app: longhorn-admission-webhook
  ports:
  - name: admission-webhook
    port: 9443
    targetPort: admission-wh
---
apiVersion: v1
kind: Service
metadata:
  labels: {{- include "longhorn.labels" . | nindent 4 }}
    app: longhorn-recovery-backend
  name: longhorn-recovery-backend
  namespace: {{ include "release_namespace" . }}
spec:
  type: ClusterIP
  sessionAffinity: ClientIP
  selector:
    app: longhorn-recovery-backend
  ports:
  - name: recovery-backend
    port: 9600
    targetPort: recov-backend
---
apiVersion: v1
kind: Service
metadata:
  labels: {{- include "longhorn.labels" . | nindent 4 }}
  name: longhorn-engine-manager
  namespace: {{ include "release_namespace" . }}
spec:
  clusterIP: None
  selector:
    longhorn.io/component: instance-manager
    longhorn.io/instance-manager-type: engine
---
apiVersion: v1
kind: Service
metadata:
  labels: {{- include "longhorn.labels" . | nindent 4 }}
  name: longhorn-replica-manager
  namespace: {{ include "release_namespace" . }}
spec:
  clusterIP: None
  selector:
    longhorn.io/component: instance-manager
    longhorn.io/instance-manager-type: replica
</file>

<file path="charts/longhorn/templates/storageclass.yaml">
apiVersion: v1
kind: ConfigMap
metadata:
  name: longhorn-storageclass
  namespace: {{ include "release_namespace" . }}
  labels: {{- include "longhorn.labels" . | nindent 4 }}
data:
  storageclass.yaml: |
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: longhorn
      annotations:
        storageclass.kubernetes.io/is-default-class: {{ .Values.persistence.defaultClass | quote }}
    provisioner: driver.longhorn.io
    allowVolumeExpansion: true
    reclaimPolicy: "{{ .Values.persistence.reclaimPolicy }}"
    volumeBindingMode: Immediate
    parameters:
      numberOfReplicas: "{{ .Values.persistence.defaultClassReplicaCount }}"
      staleReplicaTimeout: "30"
      fromBackup: ""
      {{- if .Values.persistence.defaultFsType }}
      fsType: "{{ .Values.persistence.defaultFsType }}"
      {{- end }}
      {{- if .Values.persistence.defaultMkfsParams }}
      mkfsParams: "{{ .Values.persistence.defaultMkfsParams }}"
      {{- end }}
      {{- if .Values.persistence.migratable }}
      migratable: "{{ .Values.persistence.migratable }}"
      {{- end }}    
      {{- if .Values.persistence.backingImage.enable }}
      backingImage: {{ .Values.persistence.backingImage.name }}
      backingImageDataSourceType: {{ .Values.persistence.backingImage.dataSourceType }}
      backingImageDataSourceParameters: {{ .Values.persistence.backingImage.dataSourceParameters }}
      backingImageChecksum: {{ .Values.persistence.backingImage.expectedChecksum }}
      {{- end }}
      {{- if .Values.persistence.recurringJobSelector.enable }}
      recurringJobSelector: '{{ .Values.persistence.recurringJobSelector.jobList }}'
      {{- end }}
      dataLocality: {{ .Values.persistence.defaultDataLocality | quote }}
      replicaAutoBalance: {{ .Values.persistence.defaultReplicaAutoBalance | quote }}
      {{- if .Values.persistence.defaultNodeSelector.enable }}
      nodeSelector: "{{ .Values.persistence.defaultNodeSelector.selector }}"
      {{- end }}
</file>

<file path="charts/longhorn/templates/tls-secrets.yaml">
{{- if .Values.ingress.enabled }}
{{- range .Values.ingress.secrets }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ .name }}
  namespace: {{ include "release_namespace" $ }}
  labels: {{- include "longhorn.labels" $ | nindent 4 }}
    app: longhorn
type: kubernetes.io/tls
data:
  tls.crt: {{ .certificate | b64enc }}
  tls.key: {{ .key | b64enc }}
---
{{- end }}
{{- end }}
</file>

<file path="charts/longhorn/templates/uninstall-job.yaml">
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    "helm.sh/hook": pre-delete
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
  name: longhorn-uninstall
  namespace: {{ include "release_namespace" . }}
  labels: {{- include "longhorn.labels" . | nindent 4 }}
spec:
  activeDeadlineSeconds: 900
  backoffLimit: 1
  template:
    metadata:
      name: longhorn-uninstall
      labels: {{- include "longhorn.labels" . | nindent 8 }}
    spec:
      containers:
      - name: longhorn-uninstall
        image: {{ template "registry_url" . }}{{ .Values.image.longhorn.manager.repository }}:{{ .Values.image.longhorn.manager.tag }}
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        securityContext:
          privileged: true
        command:
        - longhorn-manager
        - uninstall
        - --force
        env:
        - name: LONGHORN_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
      restartPolicy: Never
      {{- if .Values.privateRegistry.registrySecret }}
      imagePullSecrets:
      - name: {{ .Values.privateRegistry.registrySecret }}
      {{- end }}
      {{- if .Values.longhornManager.priorityClass }}
      priorityClassName: {{ .Values.longhornManager.priorityClass | quote }}
      {{- end }}
      serviceAccountName: longhorn-service-account
      {{- if or .Values.longhornManager.tolerations .Values.global.cattle.windowsCluster.enabled }}
      tolerations:
        {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.tolerations }}
{{ toYaml .Values.global.cattle.windowsCluster.tolerations | indent 6 }}
        {{- end }}
        {{- if .Values.longhornManager.tolerations }}
{{ toYaml .Values.longhornManager.tolerations | indent 6 }}
        {{- end }}
      {{- end }}
      {{- if or .Values.longhornManager.nodeSelector .Values.global.cattle.windowsCluster.enabled }}
      nodeSelector:
        {{- if and .Values.global.cattle.windowsCluster.enabled .Values.global.cattle.windowsCluster.nodeSelector }}
{{ toYaml .Values.global.cattle.windowsCluster.nodeSelector | indent 8 }}
        {{- end }}
        {{- if or .Values.longhornManager.nodeSelector }}
{{ toYaml .Values.longhornManager.nodeSelector | indent 8 }}
        {{- end }}
      {{- end }}
</file>

<file path="charts/longhorn/.helmignore">
# Patterns to ignore when building packages.
# This supports shell glob matching, relative path matching, and
# negation (prefixed with !). Only one pattern per line.
.DS_Store
# Common VCS dirs
.git/
.gitignore
.bzr/
.bzrignore
.hg/
.hgignore
.svn/
# Common backup files
*.swp
*.bak
*.tmp
*~
# Various IDEs
.project
.idea/
*.tmproj
</file>

<file path="charts/longhorn/app-readme.md">
# Longhorn

Longhorn is a lightweight, reliable and easy to use distributed block storage system for Kubernetes. Once deployed, users can leverage persistent volumes provided by Longhorn.

Longhorn creates a dedicated storage controller for each volume and synchronously replicates the volume across multiple replicas stored on multiple nodes. The storage controller and replicas are themselves orchestrated using Kubernetes. Longhorn supports snapshots, backups and even allows you to schedule recurring snapshots and backups!

**Important**: Please install Longhorn chart in `longhorn-system` namespace only.

**Warning**: Longhorn doesn't support downgrading from a higher version to a lower version.

[Chart Documentation](https://github.com/longhorn/longhorn/blob/master/chart/README.md)
</file>

<file path="charts/longhorn/Chart.yaml">
apiVersion: v1
name: longhorn
version: 1.4.0
appVersion: v1.4.0
kubeVersion: ">=1.21.0-0"
description: Longhorn is a distributed block storage system for Kubernetes.
keywords:
- longhorn
- storage
- distributed
- block
- device
- iscsi
- nfs
home: https://github.com/longhorn/longhorn
sources:
- https://github.com/longhorn/longhorn
- https://github.com/longhorn/longhorn-engine
- https://github.com/longhorn/longhorn-instance-manager
- https://github.com/longhorn/longhorn-share-manager
- https://github.com/longhorn/longhorn-manager
- https://github.com/longhorn/longhorn-ui
- https://github.com/longhorn/longhorn-tests
- https://github.com/longhorn/backing-image-manager
maintainers:
- name: Longhorn maintainers
  email: maintainers@longhorn.io
icon: https://raw.githubusercontent.com/cncf/artwork/master/projects/longhorn/icon/color/longhorn-icon-color.png
</file>

<file path="charts/longhorn/questions.yaml">
categories:
- storage
namespace: longhorn-system
questions:
- variable: image.defaultImage
  default: "true"
  description: "Use default Longhorn images"
  label: Use Default Images
  type: boolean
  show_subquestion_if: false
  group: "Longhorn Images"
  subquestions:
  - variable: image.longhorn.manager.repository
    default: longhornio/longhorn-manager
    description: "Specify Longhorn Manager Image Repository"
    type: string
    label: Longhorn Manager Image Repository
    group: "Longhorn Images Settings"
  - variable: image.longhorn.manager.tag
    default: v1.4.0
    description: "Specify Longhorn Manager Image Tag"
    type: string
    label: Longhorn Manager Image Tag
    group: "Longhorn Images Settings"
  - variable: image.longhorn.engine.repository
    default: longhornio/longhorn-engine
    description: "Specify Longhorn Engine Image Repository"
    type: string
    label: Longhorn Engine Image Repository
    group: "Longhorn Images Settings"
  - variable: image.longhorn.engine.tag
    default: v1.4.0
    description: "Specify Longhorn Engine Image Tag"
    type: string
    label: Longhorn Engine Image Tag
    group: "Longhorn Images Settings"
  - variable: image.longhorn.ui.repository
    default:  longhornio/longhorn-ui
    description: "Specify Longhorn UI Image Repository"
    type: string
    label: Longhorn UI Image Repository
    group: "Longhorn Images Settings"
  - variable: image.longhorn.ui.tag
    default: v1.4.0
    description: "Specify Longhorn UI Image Tag"
    type: string
    label: Longhorn UI Image Tag
    group: "Longhorn Images Settings"
  - variable: image.longhorn.instanceManager.repository
    default: longhornio/longhorn-instance-manager
    description: "Specify Longhorn Instance Manager Image Repository"
    type: string
    label: Longhorn Instance Manager Image Repository
    group: "Longhorn Images Settings"
  - variable: image.longhorn.instanceManager.tag
    default: v1.4.0
    description: "Specify Longhorn Instance Manager Image Tag"
    type: string
    label: Longhorn Instance Manager Image Tag
    group: "Longhorn Images Settings"
  - variable: image.longhorn.shareManager.repository
    default: longhornio/longhorn-share-manager
    description: "Specify Longhorn Share Manager Image Repository"
    type: string
    label: Longhorn Share Manager Image Repository
    group: "Longhorn Images Settings"
  - variable: image.longhorn.shareManager.tag
    default: v1.4.0
    description: "Specify Longhorn Share Manager Image Tag"
    type: string
    label: Longhorn Share Manager Image Tag
    group: "Longhorn Images Settings"
  - variable: image.longhorn.backingImageManager.repository
    default: longhornio/backing-image-manager
    description: "Specify Longhorn Backing Image Manager Image Repository"
    type: string
    label: Longhorn Backing Image Manager Image Repository
    group: "Longhorn Images Settings"
  - variable: image.longhorn.backingImageManager.tag
    default: v1.4.0
    description: "Specify Longhorn Backing Image Manager Image Tag"
    type: string
    label: Longhorn Backing Image Manager Image Tag
    group: "Longhorn Images Settings"
  - variable: image.longhorn.supportBundleKit.repository
    default: longhornio/support-bundle-kit
    description: "Specify Longhorn Support Bundle Manager Image Repository"
    type: string
    label: Longhorn Support Bundle Kit Image Repository
    group: "Longhorn Images Settings"
  - variable: image.longhorn.supportBundleKit.tag
    default: v0.0.17
    description: "Specify Longhorn Support Bundle Manager Image Tag"
    type: string
    label: Longhorn Support Bundle Kit Image Tag
    group: "Longhorn Images Settings"
  - variable: image.csi.attacher.repository
    default: longhornio/csi-attacher
    description: "Specify CSI attacher image repository. Leave blank to autodetect."
    type: string
    label: Longhorn CSI Attacher Image Repository
    group: "Longhorn CSI Driver Images"
  - variable: image.csi.attacher.tag
    default: v3.4.0
    description: "Specify CSI attacher image tag. Leave blank to autodetect."
    type: string
    label: Longhorn CSI Attacher Image Tag
    group: "Longhorn CSI Driver Images"
  - variable: image.csi.provisioner.repository
    default: longhornio/csi-provisioner
    description: "Specify CSI provisioner image repository. Leave blank to autodetect."
    type: string
    label: Longhorn CSI Provisioner Image Repository
    group: "Longhorn CSI Driver Images"
  - variable: image.csi.provisioner.tag
    default: v2.1.2
    description: "Specify CSI provisioner image tag. Leave blank to autodetect."
    type: string
    label: Longhorn CSI Provisioner Image Tag
    group: "Longhorn CSI Driver Images"
  - variable: image.csi.nodeDriverRegistrar.repository
    default: longhornio/csi-node-driver-registrar
    description: "Specify CSI Node Driver Registrar image repository. Leave blank to autodetect."
    type: string
    label: Longhorn CSI Node Driver Registrar Image Repository
    group: "Longhorn CSI Driver Images"
  - variable: image.csi.nodeDriverRegistrar.tag
    default: v2.5.0
    description: "Specify CSI Node Driver Registrar image tag. Leave blank to autodetect."
    type: string
    label: Longhorn CSI Node Driver Registrar Image Tag
    group: "Longhorn CSI Driver Images"
  - variable: image.csi.resizer.repository
    default: longhornio/csi-resizer
    description: "Specify CSI Driver Resizer image repository. Leave blank to autodetect."
    type: string
    label: Longhorn CSI Driver Resizer Image Repository
    group: "Longhorn CSI Driver Images"
  - variable: image.csi.resizer.tag
    default: v1.3.0
    description: "Specify CSI Driver Resizer image tag. Leave blank to autodetect."
    type: string
    label: Longhorn CSI Driver Resizer Image Tag
    group: "Longhorn CSI Driver Images"
  - variable: image.csi.snapshotter.repository
    default: longhornio/csi-snapshotter
    description: "Specify CSI Driver Snapshotter image repository. Leave blank to autodetect."
    type: string
    label: Longhorn CSI Driver Snapshotter Image Repository
    group: "Longhorn CSI Driver Images"
  - variable: image.csi.snapshotter.tag
    default: v5.0.1
    description: "Specify CSI Driver Snapshotter image tag. Leave blank to autodetect."
    type: string
    label: Longhorn CSI Driver Snapshotter Image Tag
    group: "Longhorn CSI Driver Images"
  - variable: image.csi.livenessProbe.repository
    default: longhornio/livenessprobe
    description: "Specify CSI liveness probe image repository. Leave blank to autodetect."
    type: string
    label: Longhorn CSI Liveness Probe Image Repository
    group: "Longhorn CSI Driver Images"
  - variable: image.csi.livenessProbe.tag
    default: v2.8.0
    description: "Specify CSI liveness probe image tag. Leave blank to autodetect."
    type: string
    label: Longhorn CSI Liveness Probe Image Tag
    group: "Longhorn CSI Driver Images"
- variable: privateRegistry.registryUrl
  label: Private registry URL
  description: "URL of private registry. Leave blank to apply system default registry."
  group: "Private Registry Settings"
  type: string
  default: ""
- variable: privateRegistry.registrySecret
  label: Private registry secret name
  description: "If create a new private registry secret is true, create a Kubernetes secret with this name; else use the existing secret of this name. Use it to pull images from your private registry."
  group: "Private Registry Settings"
  type: string
  default: ""
- variable: privateRegistry.createSecret
  default: "true"
  description: "Create a new private registry secret"
  type: boolean
  group: "Private Registry Settings"
  label: Create Secret for Private Registry Settings
  show_subquestion_if: true
  subquestions:
  - variable: privateRegistry.registryUser
    label: Private registry user
    description: "User used to authenticate to private registry."
    type: string
    default: ""
  - variable: privateRegistry.registryPasswd
    label: Private registry password
    description: "Password used to authenticate to private registry."
    type: password
    default: ""
- variable: longhorn.default_setting
  default: "false"
  description: "Customize the default settings before installing Longhorn for the first time. This option will only work if the cluster hasn't installed Longhorn."
  label: "Customize Default Settings"
  type: boolean
  show_subquestion_if: true
  group: "Longhorn Default Settings"
  subquestions:
  - variable: csi.kubeletRootDir
    default:
    description: "Specify kubelet root-dir. Leave blank to autodetect."
    type: string
    label: Kubelet Root Directory
    group: "Longhorn CSI Driver Settings"
  - variable: csi.attacherReplicaCount
    type: int
    default: 3
    min: 1
    max: 10
    description: "Specify replica count of CSI Attacher. By default 3."
    label: Longhorn CSI Attacher replica count
    group: "Longhorn CSI Driver Settings"
  - variable: csi.provisionerReplicaCount
    type: int
    default: 3
    min: 1
    max: 10
    description: "Specify replica count of CSI Provisioner. By default 3."
    label: Longhorn CSI Provisioner replica count
    group: "Longhorn CSI Driver Settings"
  - variable: csi.resizerReplicaCount
    type: int
    default: 3
    min: 1
    max: 10
    description: "Specify replica count of CSI Resizer. By default 3."
    label: Longhorn CSI Resizer replica count
    group: "Longhorn CSI Driver Settings"
  - variable: csi.snapshotterReplicaCount
    type: int
    default: 3
    min: 1
    max: 10
    description: "Specify replica count of CSI Snapshotter. By default 3."
    label: Longhorn CSI Snapshotter replica count
    group: "Longhorn CSI Driver Settings"
  - variable: defaultSettings.backupTarget
    label: Backup Target
    description: "The endpoint used to access the backupstore. NFS and S3 are supported."
    group: "Longhorn Default Settings"
    type: string
    default:
  - variable: defaultSettings.backupTargetCredentialSecret
    label: Backup Target Credential Secret
    description: "The name of the Kubernetes secret associated with the backup target."
    group: "Longhorn Default Settings"
    type: string
    default:
  - variable: defaultSettings.allowRecurringJobWhileVolumeDetached
    label: Allow Recurring Job While Volume Is Detached
    description: 'If this setting is enabled, Longhorn will automatically attaches the volume and takes snapshot/backup when it is the time to do recurring snapshot/backup.
Note that the volume is not ready for workload during the period when the volume was automatically attached. Workload will have to wait until the recurring job finishes.'
    group: "Longhorn Default Settings"
    type: boolean
    default: "false"
  - variable: defaultSettings.createDefaultDiskLabeledNodes
    label: Create Default Disk on Labeled Nodes
    description: 'Create default Disk automatically only on Nodes with the label "node.longhorn.io/create-default-disk=true" if no other disks exist. If disabled, the default disk will be created on all new nodes when each node is first added.'
    group: "Longhorn Default Settings"
    type: boolean
    default: "false"
  - variable: defaultSettings.defaultDataPath
    label: Default Data Path
    description: 'Default path to use for storing data on a host. By default "/var/lib/longhorn/"'
    group: "Longhorn Default Settings"
    type: string
    default: "/var/lib/longhorn/"
  - variable: defaultSettings.defaultDataLocality
    label: Default Data Locality
    description: 'We say a Longhorn volume has data locality if there is a local replica of the volume on the same node as the pod which is using the volume.
This setting specifies the default data locality when a volume is created from the Longhorn UI. For Kubernetes configuration, update the `dataLocality` in the StorageClass
The available modes are:
- **disabled**. This is the default option. There may or may not be a replica on the same node as the attached volume (workload)
- **best-effort**. This option instructs Longhorn to try to keep a replica on the same node as the attached volume (workload). Longhorn will not stop the volume, even if it cannot keep a replica local to the attached volume (workload) due to environment limitation, e.g. not enough disk space, incompatible disk tags, etc.'
    group: "Longhorn Default Settings"
    type: enum
    options:
    - "disabled"
    - "best-effort"
    default: "disabled"
  - variable: defaultSettings.replicaSoftAntiAffinity
    label: Replica Node Level Soft Anti-Affinity
    description: 'Allow scheduling on nodes with existing healthy replicas of the same volume. By default false.'
    group: "Longhorn Default Settings"
    type: boolean
    default: "false"
  - variable: defaultSettings.replicaAutoBalance
    label: Replica Auto Balance
    description: 'Enable this setting automatically rebalances replicas when discovered an available node.
The available global options are:
- **disabled**. This is the default option. No replica auto-balance will be done.
- **least-effort**. This option instructs Longhorn to balance replicas for minimal redundancy.
- **best-effort**. This option instructs Longhorn to balance replicas for even redundancy.
Longhorn also support individual volume setting. The setting can be specified in volume.spec.replicaAutoBalance, this overrules the global setting.
The available volume spec options are:
- **ignored**. This is the default option that instructs Longhorn to inherit from the global setting.
- **disabled**. This option instructs Longhorn no replica auto-balance should be done.
- **least-effort**. This option instructs Longhorn to balance replicas for minimal redundancy.
- **best-effort**. This option instructs Longhorn to balance replicas for even redundancy.'
    group: "Longhorn Default Settings"
    type: enum
    options:
    - "disabled"
    - "least-effort"
    - "best-effort"
    default: "disabled"
  - variable: defaultSettings.storageOverProvisioningPercentage
    label: Storage Over Provisioning Percentage
    description: "The over-provisioning percentage defines how much storage can be allocated relative to the hard drive's capacity. By default 200."
    group: "Longhorn Default Settings"
    type: int
    min: 0
    default: 200
  - variable: defaultSettings.storageMinimalAvailablePercentage
    label: Storage Minimal Available Percentage
    description: "If the minimum available disk capacity exceeds the actual percentage of available disk capacity, the disk becomes unschedulable until more space is freed up. By default 25."
    group: "Longhorn Default Settings"
    type: int
    min: 0
    max: 100
    default: 25
  - variable: defaultSettings.upgradeChecker
    label: Enable Upgrade Checker
    description: 'Upgrade Checker will check for new Longhorn version periodically. When there is a new version available, a notification will appear in the UI. By default true.'
    group: "Longhorn Default Settings"
    type: boolean
    default: "true"
  - variable: defaultSettings.defaultReplicaCount
    label: Default Replica Count
    description: "The default number of replicas when a volume is created from the Longhorn UI. For Kubernetes configuration, update the `numberOfReplicas` in the StorageClass. By default 3."
    group: "Longhorn Default Settings"
    type: int
    min: 1
    max: 20
    default: 3
  - variable: defaultSettings.defaultLonghornStaticStorageClass
    label: Default Longhorn Static StorageClass Name
    description: "The 'storageClassName' is given to PVs and PVCs that are created for an existing Longhorn volume. The StorageClass name can also be used as a label, so it is possible to use a Longhorn StorageClass to bind a workload to an existing PV without creating a Kubernetes StorageClass object. By default 'longhorn-static'."
    group: "Longhorn Default Settings"
    type: string
    default: "longhorn-static"
  - variable: defaultSettings.backupstorePollInterval
    label: Backupstore Poll Interval
    description: "In seconds. The backupstore poll interval determines how often Longhorn checks the backupstore for new backups. Set to 0 to disable the polling. By default 300."
    group: "Longhorn Default Settings"
    type: int
    min: 0
    default: 300
  - variable: defaultSettings.failedBackupTTL
    label: Failed Backup Time to Live
    description: "In minutes. This setting determines how long Longhorn will keep the backup resource that was failed. Set to 0 to disable the auto-deletion.
Failed backups will be checked and cleaned up during backupstore polling which is controlled by **Backupstore Poll Interval** setting.
Hence this value determines the minimal wait interval of the cleanup. And the actual cleanup interval is multiple of **Backupstore Poll Interval**.
Disabling **Backupstore Poll Interval** also means to disable failed backup auto-deletion."
    group: "Longhorn Default Settings"
    type: int
    min: 0
    default: 1440
  - variable: defaultSettings.restoreVolumeRecurringJobs
    label: Restore Volume Recurring Jobs
    description: "Restore recurring jobs from the backup volume on the backup target and create recurring jobs if not exist during a backup restoration.
Longhorn also supports individual volume setting. The setting can be specified on Backup page when making a backup restoration, this overrules the global setting.
The available volume setting options are:
- **ignored**. This is the default option that instructs Longhorn to inherit from the global setting.
- **enabled**. This option instructs Longhorn to restore recurring jobs/groups from the backup target forcibly.
- **disabled**. This option instructs Longhorn no restoring recurring jobs/groups should be done."
    group: "Longhorn Default Settings"
    type: boolean
    default: "false"
  - variable: defaultSettings.recurringSuccessfulJobsHistoryLimit
    label: Cronjob Successful Jobs History Limit
    description: "This setting specifies how many successful backup or snapshot job histories should be retained. History will not be retained if the value is 0."
    group: "Longhorn Default Settings"
    type: int
    min: 0
    default: 1
  - variable: defaultSettings.recurringFailedJobsHistoryLimit
    label: Cronjob Failed Jobs History Limit
    description: "This setting specifies how many failed backup or snapshot job histories should be retained. History will not be retained if the value is 0."
    group: "Longhorn Default Settings"
    type: int
    min: 0
    default: 1
  - variable: defaultSettings.supportBundleFailedHistoryLimit
    label: SupportBundle Failed History Limit
    description: "This setting specifies how many failed support bundles can exist in the cluster.
The retained failed support bundle is for analysis purposes and needs to clean up manually.
Set this value to **0** to have Longhorn automatically purge all failed support bundles."
    group: "Longhorn Default Settings"
    type: int
    min: 0
    default: 1
  - variable: defaultSettings.autoSalvage
    label: Automatic salvage
    description: "If enabled, volumes will be automatically salvaged when all the replicas become faulty e.g. due to network disconnection. Longhorn will try to figure out which replica(s) are usable, then use them for the volume. By default true."
    group: "Longhorn Default Settings"
    type: boolean
    default: "true"
  - variable: defaultSettings.autoDeletePodWhenVolumeDetachedUnexpectedly
    label: Automatically Delete Workload Pod when The Volume Is Detached Unexpectedly
    description: 'If enabled, Longhorn will automatically delete the workload pod that is managed by a controller (e.g. deployment, statefulset, daemonset, etc...) when Longhorn volume is detached unexpectedly (e.g. during Kubernetes upgrade, Docker reboot, or network disconnect). By deleting the pod, its controller restarts the pod and Kubernetes handles volume reattachment and remount.
If disabled, Longhorn will not delete the workload pod that is managed by a controller. You will have to manually restart the pod to reattach and remount the volume.
**Note:** This setting does not apply to the workload pods that do not have a controller. Longhorn never deletes them.'
    group: "Longhorn Default Settings"
    type: boolean
    default: "true"
  - variable: defaultSettings.disableSchedulingOnCordonedNode
    label: Disable Scheduling On Cordoned Node
    description: "Disable Longhorn manager to schedule replica on Kubernetes cordoned node. By default true."
    group: "Longhorn Default Settings"
    type: boolean
    default: "true"
  - variable: defaultSettings.replicaZoneSoftAntiAffinity
    label: Replica Zone Level Soft Anti-Affinity
    description: "Allow scheduling new Replicas of Volume to the Nodes in the same Zone as existing healthy Replicas. Nodes don't belong to any Zone will be treated as in the same Zone. Notice that Longhorn relies on label `topology.kubernetes.io/zone=<Zone name of the node>` in the Kubernetes node object to identify the zone. By default true."
    group: "Longhorn Default Settings"
    type: boolean
    default: "true"
  - variable: defaultSettings.nodeDownPodDeletionPolicy
    label: Pod Deletion Policy When Node is Down
    description: "Defines the Longhorn action when a Volume is stuck with a StatefulSet/Deployment Pod on a node that is down.
- **do-nothing** is the default Kubernetes behavior of never force deleting StatefulSet/Deployment terminating pods. Since the pod on the node that is down isn't removed, Longhorn volumes are stuck on nodes that are down.
- **delete-statefulset-pod** Longhorn will force delete StatefulSet terminating pods on nodes that are down to release Longhorn volumes so that Kubernetes can spin up replacement pods.
- **delete-deployment-pod** Longhorn will force delete Deployment terminating pods on nodes that are down to release Longhorn volumes so that Kubernetes can spin up replacement pods.
- **delete-both-statefulset-and-deployment-pod** Longhorn will force delete StatefulSet/Deployment terminating pods on nodes that are down to release Longhorn volumes so that Kubernetes can spin up replacement pods."
    group: "Longhorn Default Settings"
    type: enum
    options:
    - "do-nothing"
    - "delete-statefulset-pod"
    - "delete-deployment-pod"
    - "delete-both-statefulset-and-deployment-pod"
    default: "do-nothing"
  - variable: defaultSettings.allowNodeDrainWithLastHealthyReplica
    label: Allow Node Drain with the Last Healthy Replica
    description: "By default, Longhorn will block `kubectl drain` action on a node if the node contains the last healthy replica of a volume.
If this setting is enabled, Longhorn will **not** block `kubectl drain` action on a node even if the node contains the last healthy replica of a volume."
    group: "Longhorn Default Settings"
    type: boolean
    default: "false"
  - variable: defaultSettings.mkfsExt4Parameters
    label: Custom mkfs.ext4 parameters
    description: "Allows setting additional filesystem creation parameters for ext4. For older host kernels it might be necessary to disable the optional ext4 metadata_csum feature by specifying `-O ^64bit,^metadata_csum`."
    group: "Longhorn Default Settings"
    type: string
  - variable: defaultSettings.disableReplicaRebuild
    label: Disable Replica Rebuild
    description: "This setting disable replica rebuild cross the whole cluster, eviction and data locality feature won't work if this setting is true. But doesn't have any impact to any current replica rebuild and restore disaster recovery volume."
    group: "Longhorn Default Settings"
    type: boolean
    default: "false"
  - variable: defaultSettings.replicaReplenishmentWaitInterval
    label: Replica Replenishment Wait Interval
    description: "In seconds. The interval determines how long Longhorn will wait at least in order to reuse the existing data on a failed replica rather than directly creating a new replica for a degraded volume.
Warning: This option works only when there is a failed replica in the volume. And this option may block the rebuilding for a while in the case."
    group: "Longhorn Default Settings"
    type: int
    min: 0
    default: 600
  - variable: defaultSettings.concurrentReplicaRebuildPerNodeLimit
    label: Concurrent Replica Rebuild Per Node Limit
    description: "This setting controls how many replicas on a node can be rebuilt simultaneously.
Typically, Longhorn can block the replica starting once the current rebuilding count on a node exceeds the limit. But when the value is 0, it means disabling the replica rebuilding.
WARNING:
- The old setting \"Disable Replica Rebuild\" is replaced by this setting.
- Different from relying on replica starting delay to limit the concurrent rebuilding, if the rebuilding is disabled, replica object replenishment will be directly skipped.
- When the value is 0, the eviction and data locality feature won't work. But this shouldn't have any impact to any current replica rebuild and backup restore."
    group: "Longhorn Default Settings"
    type: int
    min: 0
    default: 5
  - variable: defaultSettings.concurrentVolumeBackupRestorePerNodeLimit
    label: Concurrent Volume Backup Restore Per Node Limit
    description: "This setting controls how many volumes on a node can restore the backup concurrently.
Longhorn blocks the backup restore once the restoring volume count exceeds the limit.
Set the value to **0** to disable backup restore."
    group: "Longhorn Default Settings"
    type: int
    min: 0
    default: 5
  - variable: defaultSettings.disableRevisionCounter
    label: Disable Revision Counter
    description: "This setting is only for volumes created by UI. By default, this is false meaning there will be a reivision counter file to track every write to the volume. During salvage recovering Longhorn will pick the replica with largest reivision counter as candidate to recover the whole volume. If revision counter is disabled, Longhorn will not track every write to the volume. During the salvage recovering, Longhorn will use the 'volume-head-xxx.img' file last modification time and file size to pick the replica candidate to recover the whole volume."
    group: "Longhorn Default Settings"
    type: boolean
    default: "false"
  - variable: defaultSettings.systemManagedPodsImagePullPolicy
    label: System Managed Pod Image Pull Policy
    description: "This setting defines the Image Pull Policy of Longhorn system managed pods, e.g. instance manager, engine image, CSI driver, etc. The new Image Pull Policy will only apply after the system managed pods restart."
    group: "Longhorn Default Settings"
    type: enum
    options:
    - "if-not-present"
    - "always"
    - "never"
    default: "if-not-present"
  - variable: defaultSettings.allowVolumeCreationWithDegradedAvailability
    label: Allow Volume Creation with Degraded Availability
    description: "This setting allows user to create and attach a volume that doesn't have all the replicas scheduled at the time of creation."
    group: "Longhorn Default Settings"
    type: boolean
    default: "true"
  - variable: defaultSettings.autoCleanupSystemGeneratedSnapshot
    label: Automatically Cleanup System Generated Snapshot
    description: "This setting enables Longhorn to automatically cleanup the system generated snapshot after replica rebuild is done."
    group: "Longhorn Default Settings"
    type: boolean
    default: "true"
  - variable: defaultSettings.concurrentAutomaticEngineUpgradePerNodeLimit
    label: Concurrent Automatic Engine Upgrade Per Node Limit
    description: "This setting controls how Longhorn automatically upgrades volumes' engines to the new default engine image after upgrading Longhorn manager. The value of this setting specifies the maximum number of engines per node that are allowed to upgrade to the default engine image at the same time. If the value is 0, Longhorn will not automatically upgrade volumes' engines to default version."
    group: "Longhorn Default Settings"
    type: int
    min: 0
    default: 0
  - variable: defaultSettings.backingImageCleanupWaitInterval
    label: Backing Image Cleanup Wait Interval
    description: "This interval in minutes determines how long Longhorn will wait before cleaning up the backing image file when there is no replica in the disk using it."
    group: "Longhorn Default Settings"
    type: int
    min: 0
    default: 60
  - variable: defaultSettings.backingImageRecoveryWaitInterval
    label: Backing Image Recovery Wait Interval
    description: "This interval in seconds determines how long Longhorn will wait before re-downloading the backing image file when all disk files of this backing image become failed or unknown.
    WARNING:
      - This recovery only works for the backing image of which the creation type is \"download\".
      - File state \"unknown\" means the related manager pods on the pod is not running or the node itself is down/disconnected."
    group: "Longhorn Default Settings"
    type: int
    min: 0
    default: 300
  - variable: defaultSettings.guaranteedEngineManagerCPU
    label: Guaranteed Engine Manager CPU
    description: "This integer value indicates how many percentage of the total allocatable CPU on each node will be reserved for each engine manager Pod. For example, 10 means 10% of the total CPU on a node will be allocated to each engine manager pod on this node. This will help maintain engine stability during high node workload.
    In order to prevent unexpected volume engine crash as well as guarantee a relative acceptable IO performance, you can use the following formula to calculate a value for this setting:
    Guaranteed Engine Manager CPU = The estimated max Longhorn volume engine count on a node * 0.1 / The total allocatable CPUs on the node * 100.
    The result of above calculation doesn't mean that's the maximum CPU resources the Longhorn workloads require. To fully exploit the Longhorn volume I/O performance, you can allocate/guarantee more CPU resources via this setting.
    If it's hard to estimate the usage now, you can leave it with the default value, which is 12%. Then you can tune it when there is no running workload using Longhorn volumes.
    WARNING:
      - Value 0 means unsetting CPU requests for engine manager pods.
      - Considering the possible new instance manager pods in the further system upgrade, this integer value is range from 0 to 40. And the sum with setting 'Guaranteed Engine Manager CPU' should not be greater than 40.
      - One more set of instance manager pods may need to be deployed when the Longhorn system is upgraded. If current available CPUs of the nodes are not enough for the new instance manager pods, you need to detach the volumes using the oldest instance manager pods so that Longhorn can clean up the old pods automatically and release the CPU resources. And the new pods with the latest instance manager image will be launched then.
      - This global setting will be ignored for a node if the field \"EngineManagerCPURequest\" on the node is set.
      - After this setting is changed, all engine manager pods using this global setting on all the nodes will be automatically restarted. In other words, DO NOT CHANGE THIS SETTING WITH ATTACHED VOLUMES."
    group: "Longhorn Default Settings"
    type: int
    min: 0
    max: 40
    default: 12
  - variable: defaultSettings.guaranteedReplicaManagerCPU
    label: Guaranteed Replica Manager CPU
    description: "This integer value indicates how many percentage of the total allocatable CPU on each node will be reserved for each replica manager Pod. 10 means 10% of the total CPU on a node will be allocated to each replica manager pod on this node. This will help maintain replica stability during high node workload.
    In order to prevent unexpected volume replica crash as well as guarantee a relative acceptable IO performance, you can use the following formula to calculate a value for this setting:
    Guaranteed Replica Manager CPU = The estimated max Longhorn volume replica count on a node * 0.1 / The total allocatable CPUs on the node * 100.
    The result of above calculation doesn't mean that's the maximum CPU resources the Longhorn workloads require. To fully exploit the Longhorn volume I/O performance, you can allocate/guarantee more CPU resources via this setting.
    If it's hard to estimate the usage now, you can leave it with the default value, which is 12%. Then you can tune it when there is no running workload using Longhorn volumes.
    WARNING:
      - Value 0 means unsetting CPU requests for replica manager pods.
      - Considering the possible new instance manager pods in the further system upgrade, this integer value is range from 0 to 40. And the sum with setting 'Guaranteed Replica Manager CPU' should not be greater than 40.
      - One more set of instance manager pods may need to be deployed when the Longhorn system is upgraded. If current available CPUs of the nodes are not enough for the new instance manager pods, you need to detach the volumes using the oldest instance manager pods so that Longhorn can clean up the old pods automatically and release the CPU resources. And the new pods with the latest instance manager image will be launched then.
      - This global setting will be ignored for a node if the field \"ReplicaManagerCPURequest\" on the node is set.
      - After this setting is changed, all replica manager pods using this global setting on all the nodes will be automatically restarted. In other words, DO NOT CHANGE THIS SETTING WITH ATTACHED VOLUMES."
    group: "Longhorn Default Settings"
    type: int
    min: 0
    max: 40
    default: 12
- variable: defaultSettings.kubernetesClusterAutoscalerEnabled
  label: Kubernetes Cluster Autoscaler Enabled (Experimental)
  description: "Enabling this setting will notify Longhorn that the cluster is using Kubernetes Cluster Autoscaler.
  Longhorn prevents data loss by only allowing the Cluster Autoscaler to scale down a node that met all conditions:
    - No volume attached to the node.
    - Is not the last node containing the replica of any volume.
    - Is not running backing image components pod.
    - Is not running share manager components pod."
  group: "Longhorn Default Settings"
  type: boolean
  default: false
- variable: defaultSettings.orphanAutoDeletion
  label: Orphaned Data Cleanup
  description: "This setting allows Longhorn to delete the orphan resource and its corresponding orphaned data automatically like stale replicas. Orphan resources on down or unknown nodes will not be cleaned up automatically."
  group: "Longhorn Default Settings"
  type: boolean
  default: false
- variable: defaultSettings.storageNetwork
  label: Storage Network
  description: "Longhorn uses the storage network for in-cluster data traffic. Leave this blank to use the Kubernetes cluster network.
	To segregate the storage network, input the pre-existing NetworkAttachmentDefinition in \"<namespace>/<name>\" format.
	WARNING:
	  - The cluster must have pre-existing Multus installed, and NetworkAttachmentDefinition IPs are reachable between nodes.
	  - DO NOT CHANGE THIS SETTING WITH ATTACHED VOLUMES. Longhorn will try to block this setting update when there are attached volumes.
	  - When applying the setting, Longhorn will restart all manager, instance-manager, and backing-image-manager pods."
  group: "Longhorn Default Settings"
  type: string
  default:
- variable: defaultSettings.deletingConfirmationFlag
  label: Deleting Confirmation Flag
  description: "This flag is designed to prevent Longhorn from being accidentally uninstalled which will lead to data lost.
	Set this flag to **true** to allow Longhorn uninstallation.
	If this flag **false**, Longhorn uninstallation job will fail. "
  group: "Longhorn Default Settings"
  type: boolean
  default: "false"
- variable: defaultSettings.engineReplicaTimeout
  label: Timeout between Engine and Replica
  description: "In seconds. The setting specifies the timeout between the engine and replica(s), and the value should be between 8 to 30 seconds. The default value is 8 seconds."
  group: "Longhorn Default Settings"
  type: int
  default: "8"
- variable: defaultSettings.snapshotDataIntegrity
  label: Snapshot Data Integrity
  description: "This setting allows users to enable or disable snapshot hashing and data integrity checking.
  Available options are
    - **disabled**: Disable snapshot disk file hashing and data integrity checking.
    - **enabled**: Enables periodic snapshot disk file hashing and data integrity checking. To detect the filesystem-unaware corruption caused by bit rot or other issues in snapshot disk files, Longhorn system periodically hashes files and finds corrupted ones. Hence, the system performance will be impacted during the periodical checking.
    - **fast-check**: Enable snapshot disk file hashing and fast data integrity checking. Longhorn system only hashes snapshot disk files if their are not hashed or the modification time are changed. In this mode, filesystem-unaware corruption cannot be detected, but the impact on system performance can be minimized."
  group: "Longhorn Default Settings"
  type: string
  default: "disabled"
- variable: defaultSettings.snapshotDataIntegrityImmediateCheckAfterSnapshotCreation
  label: Immediate Snapshot Data Integrity Check After Creating a Snapshot
  description: "Hashing snapshot disk files impacts the performance of the system. The immediate snapshot hashing and checking can be disabled to minimize the impact after creating a snapshot."
  group: "Longhorn Default Settings"
  type: boolean
  default: "false"
- variable: defaultSettings.snapshotDataIntegrityCronjob
  label: Snapshot Data Integrity Check CronJob
  description: "Unix-cron string format. The setting specifies when Longhorn checks the data integrity of snapshot disk files.
  Warning: Hashing snapshot disk files impacts the performance of the system. It is recommended to run data integrity checks during off-peak times and to reduce the frequency of checks."
  group: "Longhorn Default Settings"
  type: string
  default: "0 0 */7 * *"
- variable: defaultSettings.removeSnapshotsDuringFilesystemTrim
  label: Remove Snapshots During Filesystem Trim
  description: "This setting allows Longhorn filesystem trim feature to automatically mark the latest snapshot and its ancestors as removed and stops at the snapshot containing multiple children.\n\n
    Since Longhorn filesystem trim feature can be applied to the volume head and the followed continuous removed or system snapshots only.\n\n
    Notice that trying to trim a removed files from a valid snapshot will do nothing but the filesystem will discard this kind of in-memory trimmable file info.\n\n
    Later on if you mark the snapshot as removed and want to retry the trim, you may need to unmount and remount the filesystem so that the filesystem can recollect the trimmable file info."
  group: "Longhorn Default Settings"
  type: boolean
  default: "false"
- variable: defaultSettings.fastReplicaRebuildEnabled
  label: Fast Replica Rebuild Enabled
  description: "This feature supports the fast replica rebuilding. It relies on the checksum of snapshot disk files, so setting the snapshot-data-integrity to **enable** or **fast-check** is a prerequisite."
  group: "Longhorn Default Settings"
  type: boolean
  default: false
- variable: defaultSettings.replicaFileSyncHttpClientTimeout
  label: Timeout of HTTP Client to Replica File Sync Server
  description: "In seconds. The setting specifies the HTTP client timeout to the file sync server."
  group: "Longhorn Default Settings"
  type: int
  default: "30"
- variable: persistence.defaultClass
  default: "true"
  description: "Set as default StorageClass for Longhorn"
  label: Default Storage Class
  group: "Longhorn Storage Class Settings"
  required: true
  type: boolean
- variable: persistence.reclaimPolicy
  label: Storage Class Retain Policy
  description: "Define reclaim policy (Retain or Delete)"
  group: "Longhorn Storage Class Settings"
  required: true
  type: enum
  options:
  - "Delete"
  - "Retain"
  default: "Delete"
- variable: persistence.defaultClassReplicaCount
  description: "Set replica count for Longhorn StorageClass"
  label: Default Storage Class Replica Count
  group: "Longhorn Storage Class Settings"
  type: int
  min: 1
  max: 10
  default: 3
- variable: persistence.defaultDataLocality
  description: "Set data locality for Longhorn StorageClass"
  label: Default Storage Class Data Locality
  group: "Longhorn Storage Class Settings"
  type: enum
  options:
  - "disabled"
  - "best-effort"
  default: "disabled"
- variable: persistence.defaultReplicaAutoBalance
  description: "Set replica auto balance for Longhorn StorageClass"
  label: Default Storage Class Replica Auto Balance
  group: "Longhorn Storage Class Settings"
  type: enum
  options:
  - "ignored"
  - "disabled"
  - "least-effort"
  - "best-effort"
  default: "ignored"
- variable: persistence.recurringJobSelector.enable
  description: "Enable recurring job selector for Longhorn StorageClass"
  group: "Longhorn Storage Class Settings"
  label: Enable Storage Class Recurring Job Selector
  type: boolean
  default: false
  show_subquestion_if: true
  subquestions:
  - variable: persistence.recurringJobSelector.jobList
    description: 'Recurring job selector list for Longhorn StorageClass. Please be careful of quotes of input. e.g., [{"name":"backup", "isGroup":true}]'
    label: Storage Class Recurring Job Selector List
    group: "Longhorn Storage Class Settings"
    type: string
    default:
- variable: defaultSettings.defaultNodeSelector.enable
  description: "Enable recurring Node selector for Longhorn StorageClass"
  group: "Longhorn Storage Class Settings"
  label: Enable Storage Class Node Selector
  type: boolean
  default: false
  show_subquestion_if: true
  subquestions:
  - variable: defaultSettings.defaultNodeSelector.selector
    label: Storage Class Node Selector
    description: 'We use NodeSelector when we want to bind PVC via StorageClass into desired mountpoint on the nodes tagged whith its value'
    group: "Longhorn Default Settings"
    type: string
    default:
- variable: persistence.backingImage.enable
  description: "Set backing image for Longhorn StorageClass"
  group: "Longhorn Storage Class Settings"
  label: Default Storage Class Backing Image
  type: boolean
  default: false
  show_subquestion_if: true
  subquestions:
  - variable: persistence.backingImage.name
    description: 'Specify a backing image that will be used by Longhorn volumes in Longhorn StorageClass. If not exists, the backing image data source type and backing image data source parameters should be specified so that Longhorn will create the backing image before using it.'
    label: Storage Class Backing Image Name
    group: "Longhorn Storage Class Settings"
    type: string
    default:
  - variable: persistence.backingImage.expectedChecksum
    description: 'Specify the expected SHA512 checksum of the selected backing image in Longhorn StorageClass.
    WARNING:
      - If the backing image name is not specified, setting this field is meaningless.
      - It is not recommended to set this field if the data source type is \"export-from-volume\".'
    label: Storage Class Backing Image Expected SHA512 Checksum
    group: "Longhorn Storage Class Settings"
    type: string
    default:
  - variable: persistence.backingImage.dataSourceType
    description: 'Specify the data source type for the backing image used in Longhorn StorageClass.
    If the backing image does not exists, Longhorn will use this field to create a backing image. Otherwise, Longhorn will use it to verify the selected backing image.
    WARNING:
      - If the backing image name is not specified, setting this field is meaningless.
      - As for backing image creation with data source type \"upload\", it is recommended to do it via UI rather than StorageClass here. Uploading requires file data sending to the Longhorn backend after the object creation, which is complicated if you want to handle it manually.'
    label: Storage Class Backing Image Data Source Type
    group: "Longhorn Storage Class Settings"
    type: enum
    options:
    - ""
    - "download"
    - "upload"
    - "export-from-volume"
    default: ""
  - variable: persistence.backingImage.dataSourceParameters
    description: "Specify the data source parameters for the backing image used in Longhorn StorageClass.
    If the backing image does not exists, Longhorn will use this field to create a backing image. Otherwise, Longhorn will use it to verify the selected backing image.
    This option accepts a json string of a map. e.g., '{\"url\":\"https://backing-image-example.s3-region.amazonaws.com/test-backing-image\"}'.
    WARNING:
      - If the backing image name is not specified, setting this field is meaningless.
      - Be careful of the quotes here."
    label: Storage Class Backing Image Data Source Parameters
    group: "Longhorn Storage Class Settings"
    type: string
    default:
- variable: persistence.removeSnapshotsDuringFilesystemTrim
  description: "Allow automatically removing snapshots during filesystem trim for Longhorn StorageClass"
  label: Default Storage Class Remove Snapshots During Filesystem Trim
  group: "Longhorn Storage Class Settings"
  type: enum
  options:
  - "ignored"
  - "enabled"
  - "disabled"
  default: "ignored"
- variable: ingress.enabled
  default: "false"
  description: "Expose app using Layer 7 Load Balancer - ingress"
  type: boolean
  group: "Services and Load Balancing"
  label: Expose app using Layer 7 Load Balancer
  show_subquestion_if: true
  subquestions:
  - variable: ingress.host
    default: "xip.io"
    description: "layer 7 Load Balancer hostname"
    type: hostname
    required: true
    label: Layer 7 Load Balancer Hostname
  - variable: ingress.path
    default: "/"
    description: "If ingress is enabled you can set the default ingress path"
    type: string
    required: true
    label: Ingress Path
- variable: service.ui.type
  default: "Rancher-Proxy"
  description: "Define Longhorn UI service type"
  type: enum
  options:
    - "ClusterIP"
    - "NodePort"
    - "LoadBalancer"
    - "Rancher-Proxy"
  label: Longhorn UI Service
  show_if: "ingress.enabled=false"
  group: "Services and Load Balancing"
  show_subquestion_if: "NodePort"
  subquestions:
  - variable: service.ui.nodePort
    default: ""
    description: "NodePort port number(to set explicitly, choose port between 30000-32767)"
    type: int
    min: 30000
    max: 32767
    show_if: "service.ui.type=NodePort||service.ui.type=LoadBalancer"
    label: UI Service NodePort number
- variable: enablePSP
  default: "true"
  description: "Setup a pod security policy for Longhorn workloads."
  label: Pod Security Policy
  type: boolean
  group: "Other Settings"
- variable: global.cattle.windowsCluster.enabled
  default: "false"
  description: "Enable this to allow Longhorn to run on the Rancher deployed Windows cluster."
  label: Rancher Windows Cluster
  type: boolean
  group: "Other Settings"
</file>

<file path="charts/longhorn/README.md">
# Longhorn Chart

> **Important**: Please install the Longhorn chart in the `longhorn-system` namespace only.

> **Warning**: Longhorn doesn't support downgrading from a higher version to a lower version.

## Source Code

Longhorn is 100% open source software. Project source code is spread across a number of repos:

1. Longhorn Engine -- Core controller/replica logic https://github.com/longhorn/longhorn-engine
2. Longhorn Instance Manager -- Controller/replica instance lifecycle management https://github.com/longhorn/longhorn-instance-manager
3. Longhorn Share Manager -- NFS provisioner that exposes Longhorn volumes as ReadWriteMany volumes. https://github.com/longhorn/longhorn-share-manager
4. Backing Image Manager -- Backing image file lifecycle management. https://github.com/longhorn/backing-image-manager
5. Longhorn Manager -- Longhorn orchestration, includes CSI driver for Kubernetes https://github.com/longhorn/longhorn-manager
6. Longhorn UI -- Dashboard https://github.com/longhorn/longhorn-ui

## Prerequisites

1. A container runtime compatible with Kubernetes (Docker v1.13+, containerd v1.3.7+, etc.)
2. Kubernetes v1.18+
3. Make sure `bash`, `curl`, `findmnt`, `grep`, `awk` and `blkid` has been installed in all nodes of the Kubernetes cluster.
4. Make sure `open-iscsi` has been installed, and the `iscsid` daemon is running on all nodes of the Kubernetes cluster. For GKE, recommended Ubuntu as guest OS image since it contains `open-iscsi` already.

## Installation
1. Add Longhorn chart repository.
```
helm repo add longhorn https://charts.longhorn.io
```

2. Update local Longhorn chart information from chart repository.
```
helm repo update
```

3. Install Longhorn chart.
- With Helm 2, the following command will create the `longhorn-system` namespace and install the Longhorn chart together.
```
helm install longhorn/longhorn --name longhorn --namespace longhorn-system
``` 
- With Helm 3, the following commands will create the `longhorn-system` namespace first, then install the Longhorn chart.

```
kubectl create namespace longhorn-system
helm install longhorn longhorn/longhorn --namespace longhorn-system
```

## Uninstallation

With Helm 2 to uninstall Longhorn.
```
helm delete longhorn --purge
```

With Helm 3 to uninstall Longhorn.
```
helm uninstall longhorn -n longhorn-system
kubectl delete namespace longhorn-system
```

---
Please see [link](https://github.com/longhorn/longhorn) for more information.
</file>

<file path="charts/longhorn/values.yaml">
# Default values for longhorn.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.
global:
  cattle:
    systemDefaultRegistry: ""
    windowsCluster:
      # Enable this to allow Longhorn to run on the Rancher deployed Windows cluster
      enabled: false
      # Tolerate Linux node taint
      tolerations:
      - key: "cattle.io/os"
        value: "linux"
        effect: "NoSchedule"
        operator: "Equal"
      # Select Linux nodes
      nodeSelector:
        kubernetes.io/os: "linux"
      # Recognize toleration and node selector for Longhorn run-time created components
      defaultSetting:
        taintToleration: cattle.io/os=linux:NoSchedule
        systemManagedComponentsNodeSelector: kubernetes.io/os:linux

image:
  longhorn:
    engine:
      repository: longhornio/longhorn-engine
      tag: v1.4.0
    manager:
      repository: longhornio/longhorn-manager
      tag: v1.4.0
    ui:
      repository: longhornio/longhorn-ui
      tag: v1.4.0
    instanceManager:
      repository: longhornio/longhorn-instance-manager
      tag: v1.4.0
    shareManager:
      repository: longhornio/longhorn-share-manager
      tag: v1.4.0
    backingImageManager:
      repository: longhornio/backing-image-manager
      tag: v1.4.0
    supportBundleKit:
      repository: longhornio/support-bundle-kit
      tag: v0.0.17
  csi:
    attacher:
      repository: longhornio/csi-attacher
      tag: v3.4.0
    provisioner:
      repository: longhornio/csi-provisioner
      tag: v2.1.2
    nodeDriverRegistrar:
      repository: longhornio/csi-node-driver-registrar
      tag: v2.5.0
    resizer:
      repository: longhornio/csi-resizer
      tag: v1.3.0
    snapshotter:
      repository: longhornio/csi-snapshotter
      tag: v5.0.1
    livenessProbe:
      repository: longhornio/livenessprobe
      tag: v2.8.0
  pullPolicy: IfNotPresent

service:
  ui:
    type: ClusterIP
    nodePort: null
  manager:
    type: ClusterIP
    nodePort: ""
    loadBalancerIP: ""
    loadBalancerSourceRanges: ""

persistence:
  defaultClass: true
  defaultFsType: ext4
  defaultMkfsParams: ""
  defaultClassReplicaCount: 3
  defaultDataLocality: disabled # best-effort otherwise
  defaultReplicaAutoBalance: ignored # "disabled", "least-effort" or "best-effort" otherwise
  reclaimPolicy: Delete
  migratable: false
  recurringJobSelector:
    enable: false
    jobList: []
  backingImage:
    enable: false
    name: ~
    dataSourceType: ~
    dataSourceParameters: ~
    expectedChecksum: ~
  defaultNodeSelector:
    enable: false # disable by default
    selector: []
  removeSnapshotsDuringFilesystemTrim: ignored # "enabled" or "disabled" otherwise

csi:
  kubeletRootDir: ~
  attacherReplicaCount: ~
  provisionerReplicaCount: ~
  resizerReplicaCount: ~
  snapshotterReplicaCount: ~

defaultSettings:
  backupTarget: ~
  backupTargetCredentialSecret: ~
  allowRecurringJobWhileVolumeDetached: ~
  createDefaultDiskLabeledNodes: ~
  defaultDataPath: ~
  defaultDataLocality: ~
  replicaSoftAntiAffinity: ~
  replicaAutoBalance: ~
  storageOverProvisioningPercentage: ~
  storageMinimalAvailablePercentage: ~
  upgradeChecker: ~
  defaultReplicaCount: ~
  defaultLonghornStaticStorageClass: ~
  backupstorePollInterval: ~
  failedBackupTTL: ~
  restoreVolumeRecurringJobs: ~
  recurringSuccessfulJobsHistoryLimit: ~
  recurringFailedJobsHistoryLimit: ~
  supportBundleFailedHistoryLimit: ~
  taintToleration: ~
  systemManagedComponentsNodeSelector: ~
  priorityClass: ~
  autoSalvage: ~
  autoDeletePodWhenVolumeDetachedUnexpectedly: ~
  disableSchedulingOnCordonedNode: ~
  replicaZoneSoftAntiAffinity: ~
  nodeDownPodDeletionPolicy: ~
  allowNodeDrainWithLastHealthyReplica: ~
  mkfsExt4Parameters: ~
  disableReplicaRebuild: ~
  replicaReplenishmentWaitInterval: ~
  concurrentReplicaRebuildPerNodeLimit: ~
  concurrentVolumeBackupRestorePerNodeLimit: ~
  disableRevisionCounter: ~
  systemManagedPodsImagePullPolicy: ~
  allowVolumeCreationWithDegradedAvailability: ~
  autoCleanupSystemGeneratedSnapshot: ~
  concurrentAutomaticEngineUpgradePerNodeLimit: ~
  backingImageCleanupWaitInterval: ~
  backingImageRecoveryWaitInterval: ~
  guaranteedEngineManagerCPU: ~
  guaranteedReplicaManagerCPU: ~
  kubernetesClusterAutoscalerEnabled: ~
  orphanAutoDeletion: ~
  storageNetwork: ~
  deletingConfirmationFlag: ~
  engineReplicaTimeout: ~
  snapshotDataIntegrity: ~
  snapshotDataIntegrityImmediateCheckAfterSnapshotCreation: ~
  snapshotDataIntegrityCronjob: ~
  removeSnapshotsDuringFilesystemTrim: ~
  fastReplicaRebuildEnabled: ~
  replicaFileSyncHttpClientTimeout: ~
privateRegistry:
  createSecret: ~
  registryUrl: ~
  registryUser: ~
  registryPasswd: ~
  registrySecret: ~

longhornManager:
  log:
    ## Allowed values are `plain` or `json`.
    format: plain
  priorityClass: ~
  tolerations: []
  ## If you want to set tolerations for Longhorn Manager DaemonSet, delete the `[]` in the line above
  ## and uncomment this example block
  # - key: "key"
  #   operator: "Equal"
  #   value: "value"
  #   effect: "NoSchedule"
  nodeSelector: {}
  ## If you want to set node selector for Longhorn Manager DaemonSet, delete the `{}` in the line above
  ## and uncomment this example block
  #  label-key1: "label-value1"
  #  label-key2: "label-value2"
  serviceAnnotations: {}
  ## If you want to set annotations for the Longhorn Manager service, delete the `{}` in the line above
  ## and uncomment this example block
  #  annotation-key1: "annotation-value1"
  #  annotation-key2: "annotation-value2"

longhornDriver:
  priorityClass: ~
  tolerations: []
  ## If you want to set tolerations for Longhorn Driver Deployer Deployment, delete the `[]` in the line above
  ## and uncomment this example block
  # - key: "key"
  #   operator: "Equal"
  #   value: "value"
  #   effect: "NoSchedule"
  nodeSelector: {}
  ## If you want to set node selector for Longhorn Driver Deployer Deployment, delete the `{}` in the line above
  ## and uncomment this example block
  #  label-key1: "label-value1"
  #  label-key2: "label-value2"

longhornUI:
  replicas: 2
  priorityClass: ~
  tolerations: []
  ## If you want to set tolerations for Longhorn UI Deployment, delete the `[]` in the line above
  ## and uncomment this example block
  # - key: "key"
  #   operator: "Equal"
  #   value: "value"
  #   effect: "NoSchedule"
  nodeSelector: {}
  ## If you want to set node selector for Longhorn UI Deployment, delete the `{}` in the line above
  ## and uncomment this example block
  #  label-key1: "label-value1"
  #  label-key2: "label-value2"

longhornConversionWebhook:
  replicas: 2
  priorityClass: ~
  tolerations: []
  ## If you want to set tolerations for Longhorn conversion webhook Deployment, delete the `[]` in the line above
  ## and uncomment this example block
  # - key: "key"
  #   operator: "Equal"
  #   value: "value"
  #   effect: "NoSchedule"
  nodeSelector: {}
  ## If you want to set node selector for Longhorn conversion webhook Deployment, delete the `{}` in the line above
  ## and uncomment this example block
  #  label-key1: "label-value1"
  #  label-key2: "label-value2"

longhornAdmissionWebhook:
  replicas: 2
  priorityClass: ~
  tolerations: []
  ## If you want to set tolerations for Longhorn admission webhook Deployment, delete the `[]` in the line above
  ## and uncomment this example block
  # - key: "key"
  #   operator: "Equal"
  #   value: "value"
  #   effect: "NoSchedule"
  nodeSelector: {}
  ## If you want to set node selector for Longhorn admission webhook Deployment, delete the `{}` in the line above
  ## and uncomment this example block
  #  label-key1: "label-value1"
  #  label-key2: "label-value2"

longhornRecoveryBackend:
  replicas: 2
  priorityClass: ~
  tolerations: []
  ## If you want to set tolerations for Longhorn recovery backend Deployment, delete the `[]` in the line above
  ## and uncomment this example block
  # - key: "key"
  #   operator: "Equal"
  #   value: "value"
  #   effect: "NoSchedule"
  nodeSelector: {}
  ## If you want to set node selector for Longhorn recovery backend Deployment, delete the `{}` in the line above
  ## and uncomment this example block
  #  label-key1: "label-value1"
  #  label-key2: "label-value2"

ingress:
  ## Set to true to enable ingress record generation
  enabled: false

  ## Add ingressClassName to the Ingress
  ## Can replace the kubernetes.io/ingress.class annotation on v1.18+
  ingressClassName: ~

  host: sslip.io

  ## Set this to true in order to enable TLS on the ingress record
  tls: false

  ## Enable this in order to enable that the backend service will be connected at port 443
  secureBackends: false

  ## If TLS is set to true, you must declare what secret will store the key/certificate for TLS
  tlsSecret: longhorn.local-tls

  ## If ingress is enabled you can set the default ingress path
  ## then you can access the UI by using the following full path {{host}}+{{path}}
  path: /

  ## Ingress annotations done as key:value pairs
  ## If you're using kube-lego, you will want to add:
  ## kubernetes.io/tls-acme: true
  ##
  ## For a full list of possible ingress annotations, please see
  ## ref: https://github.com/kubernetes/ingress-nginx/blob/master/docs/annotations.md
  ##
  ## If tls is set to true, annotation ingress.kubernetes.io/secure-backends: "true" will automatically be set
  annotations:
  #  kubernetes.io/ingress.class: nginx
  #  kubernetes.io/tls-acme: true

  secrets:
  ## If you're providing your own certificates, please use this to add the certificates as secrets
  ## key and certificate should start with -----BEGIN CERTIFICATE----- or
  ## -----BEGIN RSA PRIVATE KEY-----
  ##
  ## name should line up with a tlsSecret set further up
  ## If you're using kube-lego, this is unneeded, as it will create the secret for you if it is not set
  ##
  ## It is also possible to create and manage the certificates outside of this helm chart
  ## Please see README.md for more information
  # - name: longhorn.local-tls
  #   key:
  #   certificate:

#  For Kubernetes < v1.25, if your cluster enables Pod Security Policy admission controller,
#  set this to `true` to ship longhorn-psp which allow privileged Longhorn pods to start
enablePSP: false

## Specify override namespace, specifically this is useful for using longhorn as sub-chart
## and its release namespace is not the `longhorn-system`
namespaceOverride: ""

# Annotations to add to the Longhorn Manager DaemonSet Pods. Optional.
annotations: {}

serviceAccount:
  # Annotations to add to the service account
  annotations: {}
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

<file path="README.md">
# Longhorn Charts
![Release Charts](https://github.com/longhorn/charts/workflows/Release%20Charts/badge.svg)

This repository contains the charts used for installing Longhorn using Helm. Currently, this chart only contains the following chart:
- `longhorn` - The chart for Longhorn. For the actual Longhorn repository, go [here](https://github.com/longhorn/longhorn).

## Adding the Chart
```
$ helm repo add longhorn https://charts.longhorn.io
$ helm repo update
```

## Releasing New Charts
When ready to release a new chart version or add a new chart, copy the chart directory from the source repository into the `charts/` directory. Make sure the chart directory is named after the actual chart (for example: `longhorn/`).

Once pushed, GitHub Actions will look for any changes to charts in the `charts/` directory since the last tagged release in the repository, package any changed charts, and then release them on `GitHub Pages`.

Note that changes should only be synced to this repository when ready for a new release. GitHub Actions will fail if making changes to the charts in this repository directly, as Chart Releaser will not attempt to override old chart releases.
</file>

</files>
