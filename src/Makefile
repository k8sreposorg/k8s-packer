PACKER_BINARY ?= packer
PACKER_VARIABLES := aws_region ami_name binary_bucket_name binary_bucket_region kubernetes_version kubernetes_build_date kernel_version docker_version containerd_version cni_plugin_version source_ami_id source_ami_owners arch instance_type security_group_id additional_yum_repos pull_cni_from_github sonobuoy_e2e_registry

K8S_VERSION_PARTS := $(subst ., ,$(kubernetes_version))
K8S_VERSION_MINOR := $(word 1,${K8S_VERSION_PARTS}).$(word 2,${K8S_VERSION_PARTS})

aws_region ?= $(AWS_DEFAULT_REGION)
binary_bucket_region ?= $(AWS_DEFAULT_REGION)
arch ?= x86_64

ifeq ($(arch), arm64)
instance_type ?= m6g.large
ami_name ?= amazon-eks-arm64-node-$(K8S_VERSION_MINOR)-v$(shell date +'%Y%m%d')
else
instance_type ?= m4.large
ami_name ?= amazon-eks-node-$(K8S_VERSION_MINOR)-v$(shell date +'%Y%m%d')
endif

.PHONY: all
all: 1.18 

.PHONY: validate
validate:
	$(PACKER_BINARY) validate $(foreach packerVar,$(PACKER_VARIABLES), $(if $($(packerVar)),--var $(packerVar)='$($(packerVar))',)) eks-worker-node.json

.PHONY: k8s
k8s: validate
	@echo "$(T_GREEN)Building AMI for version $(T_YELLOW)$(kubernetes_version)$(T_GREEN) on $(T_YELLOW)$(arch)$(T_RESET)"
	$(PACKER_BINARY) build $(foreach packerVar,$(PACKER_VARIABLES), $(if $($(packerVar)),--var $(packerVar)='$($(packerVar))',)) eks-worker-node.json

# Build dates and versions taken from https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html



.PHONY: 1.18
1.18:
	$(MAKE) k8s kubernetes_version=1.18.9 kubernetes_build_date=2020-11-02 pull_cni_from_github=true
