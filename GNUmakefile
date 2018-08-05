
-include config.unix

LUA_ENV_VARS = LUA_INIT LUA_PATH LUA_PATH_5_2 LUA_PATH_5_3 LUA_PATH_5_4 LUA_CPATH LUA_CPATH_5_2 LUA_CPATH_5_3 LUA_CPATH_5_4

# See https://www.gnu.org/software/make/manual/html_node/Makefile-Conventions.html
prefix ?= /usr/local
datarootdir ?= $(prefix)/share
bindir ?= $(prefix)/bin
sysconfdir ?= $(prefix)/etc
INSTALL ?= install
INSTALL_DATA ?= $(INSTALL) -m 644

LUA_INTERPRETER ?= lua
ifdef LUA_BINDIR
LUA = $(LUA_BINDIR)/$(LUA_INTERPRETER)
SHEBANG = \#!$(LUA_BINDIR)/$(LUA_INTERPRETER)
else
LUA = $(LUA_INTERPRETER)
SHEBANG = \#!/usr/bin/env $(LUA_INTERPRETER)
endif
LUA_VERSION ?= $(shell unset $(LUA_ENV_VARS); $(LUA) -e 'print(_VERSION:match(" (5%.[1234])$$"))')
rocks_tree ?= $(prefix)
luarocksconfdir ?= $(sysconfdir)/luarocks
luadir ?= $(datarootdir)/lua/$(LUA_VERSION)


LUAROCKS_FILES = $(shell find src/luarocks/ -type f -name '*.lua')

all: build

# ----------------------------------------
# Base build
# ----------------------------------------

build: ./build/luarocks ./build/luarocks-admin

config-$(LUA_VERSION).lua.in:
	@printf -- '-- LuaRocks configuration\n\n'\
	'rocks_trees = {\n'\
	'   { name = "user", root = home .. "/.luarocks" };\n'\
	'   { name = "system", root = "'"$(rocks_tree)"'" };\n'\
	'}\n'\
	"$$([ -n "$(LUA_INTERPRETER)" ] && printf 'lua_interpreter = "%s";\\n' "$(LUA_INTERPRETER)")"\
	'variables = {\n'\
	"$$([ -n "$(LUA_DIR)" ] && printf '   LUA_DIR = "%s";\\n' "$(LUA_DIR)")"\
	"$$([ -n "$(LUA_INCDIR)" ] && printf '   LUA_INCDIR = "%s";\\n' "$(LUA_INCDIR)")"\
	"$$([ -n "$(LUA_BINDIR)" ] && printf '   LUA_BINDIR = "%s";\\n' "$(LUA_BINDIR)")"\
	"$$([ -n "$(LUA_LIBDIR)" ] && printf '   LUA_LIBDIR = "%s";\\n' "$(LUA_LIBDIR)")"\
	'}\n'\
	> $@

./build/luarocks: src/bin/luarocks
	mkdir -p "$(@D)"
	(printf '$(SHEBANG)\n'\
	'package.loaded["luarocks.core.hardcoded"] = { SYSCONFDIR = [[$(luarocksconfdir)]] }\n'\
	'package.path=[[$(luadir)/?.lua;]] .. package.path\n'; \
	tail -n +2 src/bin/luarocks \
	)> "$@"

./build/luarocks-admin: src/bin/luarocks-admin
	mkdir -p "$(@D)"
	(printf '$(SHEBANG)\n'\
	'package.loaded["luarocks.core.hardcoded"] = { SYSCONFDIR = [[$(luarocksconfdir)]] }\n'\
	'package.path=[[$(luadir)/?.lua;]] .. package.path\n'; \
	tail -n +2 src/bin/luarocks-admin \
	)> "$@"

# ----------------------------------------
# Regular install
# ----------------------------------------

INSTALL_FILES = $(DESTDIR)$(bindir)/luarocks \
	$(DESTDIR)$(bindir)/luarocks-admin \
	$(DESTDIR)$(luarocksconfdir)/config-$(LUA_VERSION).lua \
	$(patsubst src/%, $(DESTDIR)$(luadir)/%, $(LUAROCKS_FILES))

install: $(INSTALL_FILES)

$(DESTDIR)$(bindir)/luarocks: ./build/luarocks
	$(INSTALL) -D "$<" "$@"

$(DESTDIR)$(bindir)/luarocks-admin: ./build/luarocks-admin
	$(INSTALL) -D "$<" "$@"

$(DESTDIR)$(luadir)/luarocks/%.lua: src/luarocks/%.lua
	$(INSTALL_DATA) -D "$<" "$@"

$(DESTDIR)$(luarocksconfdir)/config-$(LUA_VERSION).lua: config-$(LUA_VERSION).lua.in
	$(INSTALL_DATA) -D "$<" "$@"

uninstall:
	rm -rf $(INSTALL_FILES)

# ----------------------------------------
# Binary build
# ----------------------------------------

binary: build-binary/luarocks.exe build-binary/luarocks-admin.exe

build-binary/luarocks.exe: luarocks
	LUA_PATH="$(CURDIR)/src/?.lua;;" "$(LUA_BINDIR)/$(LUA_INTERPRETER)" binary/all_in_one "src/bin/luarocks" "$(LUA_DIR)" "^src/luarocks/admin/" "$(luarocksconfdir)" build-binary $(BINARY_PLATFORM) $(BINARY_CC) $(BINARY_NM) $(BINARY_SYSROOT)

build-binary/luarocks-admin.exe: luarocks
	LUA_PATH="$(CURDIR)/src/?.lua;;" "$(LUA_BINDIR)/$(LUA_INTERPRETER)" binary/all_in_one "src/bin/luarocks-admin" "$(LUA_DIR)" "^src/luarocks/cmd/" "$(luarocksconfdir)" build-binary $(BINARY_PLATFORM) $(BINARY_CC) $(BINARY_NM) $(BINARY_SYSROOT)

# ----------------------------------------
# Binary install
# ----------------------------------------

install-binary: build-binary/luarocks.exe build-binary/luarocks-admin.exe
	mkdir -p "$(DESTDIR)$(bindir)"
	cp build-binary/luarocks.exe "$(DESTDIR)$(bindir)/luarocks"
	chmod +rx "$(DESTDIR)$(bindir)/luarocks"
	cp build-binary/luarocks-admin.exe "$(DESTDIR)$(bindir)/luarocks-admin"
	chmod +rx "$(DESTDIR)$(bindir)/luarocks-admin"
	mkdir -p "$(DESTDIR)$(luadir)/luarocks/core"
	cp -a src/luarocks/core/* "$(DESTDIR)$(luadir)/luarocks/core"
	cp -a src/luarocks/loader.lua "$(DESTDIR)$(luadir)/luarocks/"

# ----------------------------------------
# Bootstrap install
# ----------------------------------------

bootstrap: $(DESTDIR)$(luarocksconfdir)/config-$(LUA_VERSION).lua
	(unset $(LUA_ENV_VARS); \
	LUA_PATH="./src/?.lua;;" \
	LUAROCKS_SYSCONFDIR="$(DESTDIR)$(luarocksconfdir)" \
	"$(LUA)" ./src/bin/luarocks make --tree="$(DESTDIR)$(rocks_tree)")

# ----------------------------------------
# Windows binary build
# ----------------------------------------

windows-binary: luarocks
	$(MAKE) -f binary/Makefile.windows windows-binary

windows-clean:
	$(MAKE) -f binary/Makefile.windows windows-clean

# ----------------------------------------
# Clean
# ----------------------------------------

clean: windows-clean
	rm -rf ./config.unix \
		./build/ \
		build-binary \
		./.luarocks \
		./lua_modules

.PHONY: all build install binary install-binary bootstrap clean windows-binary windows-clean
