DEPS = install golang.x86_64 criu.x86_64 squashfs-tools.x86_64
RELEASE = lxc-2.5
PKG_MGR = yum

export GOPATH = $(HOME)/go
SRC_DIR = $(GOPATH)/src/github.com/lxc/lxd

# packaging
SERVICES = /lib/systemd/system/lxd.service /lib/systemd/system/dnsmasq@.service
BINARIES =  /usr/bin/lxc /usr/bin/lxd
BRIDGE = br0
DNSMASQCONF = /etc/dnsmasq-$(BRIDGE).conf

.PHONY: all src deps build install groups enable

build: src
	export GOPATH=$(GOPATH); $(MAKE) -C $(SRC_DIR)

# prepare deps and grab src code only
src: deps $(SRC_DIR)
	@echo "run 'make build' to compile"

deps:
	sudo $(PKG_MGR) install -y $(DEPS)

$(SRC_DIR):
	@echo "GOPATH: $(GOPATH)"
	mkdir -p $(GOPATH)
	export GOPATH=$(GOPATH); go get github.com/lxc/lxd

install: groups $(BINARIES) $(SERVICES)
	@echo "all done!"

# add lxd unix group and
groups:
	groupadd --system lxd || :
	usermod -a -G lxd ${USER}
	useradd dnsmasq --system

# enable services
enable: $(SERVICES) $(DNSMASQCONF)
	systemctl enable lxd.service
	systemctl start lxd.service
	systemctl enable dnsmasq@$(BRIDGE).service
	systemctl start dnsmasq@$(BRIDGE).service

# install systemd unit files
$(SERVICES):
	install -m 0755 $(@F) $@

# install dnsmasq configuration
$(DNSMASQCONF):
	install -o dnsmasq -g dnsmasq -m 0644 $(@F) $@

all: src build install

# install lxd in system path
$(BINARIES):
	echo $$GOPATH
	install -m 0755 --owner=root --group=lxd  $(GOPATH)/bin/$(@F) $@
