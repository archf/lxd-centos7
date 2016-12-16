
DEPS = install golang.x86_64 criu.x86_64 squashfs-tools.x86_64
RELEASE = lxc-2.5
PKG_MGR = yum

export GOPATH = $(HOME)/go
# export GOPATH
SRC_DIR = $(GOPATH)/src/github.com/lxc/lxd

# packaging
SERVICES = /lib/systemd/system/lxd.service /lib/systemd/system/dnsmasq@.service
BINARIES =  /usr/bin/lxc /usr/bin/lxd
BRIDGE = br0
DNSMASQCONF = /etc/dnsmasq-$(BRIDGE).conf

.PHONY: all src deps build install group enable

# prepare deps and grab src code only
src: deps $(SRC_DIR)
	@echo "run 'make build' to compile"

build: src
	export GOPATH=$(GOPATH); $(MAKE) -C $(SRC_DIR)

all: src build install

group:
	groupadd --system lxd || :
	usermod -a -G lxd ${USER}
	useradd dnsmasq --system

install: group $(BINARIES) $(SERVICES)
	@echo "all done!"

$(BINARIES):
	echo $$GOPATH
	install -m 0755 --owner=root --group=lxd  $(GOPATH)/bin/$(@F) $@

$(SERVICES):
	install -m 0755 $(@F) $@

enable: $(SERVICES) $(DNSMASQCONF)
	systemctl enable lxd.service
	systemctl start lxd.service
	systemctl enable dnsmasq@$(BRIDGE).service
	systemctl start dnsmasq@$(BRIDGE).service

$(DNSMASQCONF):
	install -o dnsmasq -g dnsmasq -m 0644 $(@F) $@

deps:
	sudo $(PKG_MGR) install -y $(DEPS)

$(SRC_DIR):
	@echo "GOPATH: $(GOPATH)"
	mkdir -p $(GOPATH)
	export GOPATH=$(GOPATH); go get github.com/lxc/lxd
