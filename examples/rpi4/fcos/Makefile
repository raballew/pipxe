COREOS_ASSEMBLER_VERSION				:= quay.io/coreos-assembler/coreos-assembler:latest
COREOS_ASSEMBLER_CONFIG_GIT				:= ./fedora-coreos-config
COREOS_ASSEMBLER_GIT					:= ./coreos-assembler
COREOS_ASSEMBLER_CONTAINER_RUNTIME_ARGS	:= 
COREOS_BUTANE_VERSION					:= quay.io/coreos/butane:release
BUILD_DIR								:= ./target


all : clean cosa setup dts kernel ignition

define cosa
	podman run --rm -ti --security-opt label=disable --privileged \
		--uidmap=1000:0:1 --uidmap=0:1:1000 --uidmap 1001:1001:64536 \
		-v $(BUILD_DIR):/srv/ --device /dev/kvm --device /dev/fuse \
		--tmpfs /tmp -v /var/tmp:/var/tmp --name cosa \
		-v $(COREOS_ASSEMBLER_CONFIG_GIT):/srv/src/config/:ro \
		-v $(COREOS_ASSEMBLER_GIT)/src/:/usr/lib/coreos-assembler/:ro \
		$(COREOS_ASSEMBLER_CONTAINER_RUNTIME_ARGS) \
		$(COREOS_ASSEMBLER_VERSION)
endef

clean :
	$(call cosa) prune
	$(call cosa) clean
	podman container rm cosa
	podman image rm $(COREOS_ASSEMBLER_VERSION) $(COREOS_BUTANE_VERSION)
	rm -rf $(BUILD_DIR)

cosa :
# build aarch64 cosa	

setup : 
	podman pull $(COREOS_ASSEMBLER_VERSION) $(COREOS_BUTANE_VERSION)
	mkdir -p $(BUILD_DIR)
	$(call cosa) init --force /dev/null/

dts :


kernel :
	$(call cosa) fetch
	$(call cosa) build

ignition : 
	podman run --pull=always --rm -v ${PWD}:/data:z -w /data $(COREOS_BUTANE_VERSION) --pretty --strict -d /data/ fcos.bu > fcos.ign

.PHONY : clean cosa setup dts kernel ignition
