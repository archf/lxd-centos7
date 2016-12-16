SRC_DIR = lxc
SRC = https://github.com/lxc/lxc.git
RELEASE = lxc-2.0.6
PKG_MGR = yum
# LDFLAGS = "-L/usr/lib"

CONFIGURE_FLAGS = --prefix=/usr --with-distro=centos --libdir=/usr/lib64 \
									--sysconfdir=/etc --mandir=/usr/share/man \
									--localstatedir=/var --enable-doc \
									--enable-tests --enable-examples

DEPS = python-devel doxygen.x86_64 graphviz-devel.x86_64 graphviz.x86_64 \
			 libcap-devel libcgroup docbook2X.x86_64

.PHONY: all src deps build install

# prepare deps and grab src code only
src: deps $(SRC_DIR)

all: deps $(SRC_DIR) build install

install:
	sudo $(MAKE) -C $(SRC_DIR) install

deps:
	sudo $(PKG_MGR) install -y $(DEPS)

$(SRC_DIR):
	git clone --depth 1 --branch $(RELEASE) $(SRC)

build: configure
	cd $(SRC_DIR) ; ./configure $(CONFIGURE_FLAGS)  LDFLAGS=$(LDFLAGS)
	$(MAKE) -C $(SRC_DIR)

configure:
	cd $(SRC_DIR) ; ./autogen.sh --libdir=/usr/lib64
