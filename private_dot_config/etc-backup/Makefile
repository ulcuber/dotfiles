SOURCEDIR := /etc
BACKUPDIR := $(CURDIR)/files

FILES := \
	nginx/octane-failover-example.conf \
	nginx/nuxt-failover-example.conf \
	udev/rules.d/99-bluetooth-keyboard.rules \
	udev/rules.d/99-hdmi.rules \
	udev/rules.d/99-KBW-6NX.0_Keyboard.rules \
	udev/rules.d/99-uinput.rules \
	init.d/kmonad \
	init.d/kmonad-bluetooth-3.0-keyboard \
	init.d/kmonad-kbw-6nx.0-keyboard \
	init.d/AmneziaVPNd \
	X11/xorg.conf.d/00-keyboard.conf \
	X11/xorg.conf.d/40-libinput.conf \
	mysql/mysql.d/90-user.cnf \
	portage/make.conf

.PHONY: backup install clean list push pull

backup:
	@echo "Backing up files..."
	@for file in $(FILES); do \
		src="$(SOURCEDIR)/$$file"; \
		dst="$(BACKUPDIR)/$$file"; \
		if [ -f "$$src" ]; then \
			mkdir -p $$(dirname $$dst); \
			cp $$src $$dst; \
			echo "  backed up: $$file"; \
		fi; \
	done

install:
	@echo "Installing files (preserving existing)..."
	@for file in $(FILES); do \
		src="$(BACKUPDIR)/$$file"; \
		dst="$(SOURCEDIR)/$$file"; \
		dst_dir="$$(dirname $$dst)"; \
		if [ -f "$$src" ]; then \
			if [ ! -d "$$dst_dir" ]; then \
				echo "  skipped (no dir): $$dst"; \
			elif [ -f "$$dst" ]; then \
				echo "  skipped (exists): $$file"; \
			else \
				sudo cp $$src $$dst; \
				echo "  installed: $$file"; \
			fi \
		else \
			echo "  skipped (no backup): $$file"; \
		fi; \
	done

list:
	@echo "Source files:"
	@for file in $(FILES); do \
		if [ -f "$(SOURCEDIR)/$$file" ]; then \
			echo "  [present] $$file"; \
		else \
			echo "  [missing] $$file"; \
		fi; \
	done
	@echo "Backup files:"
	@for file in $(FILES); do \
		if [ -f "$(BACKUPDIR)/$$file" ]; then \
			echo "  [present] $$file"; \
		else \
			echo "  [missing] $$file"; \
		fi; \
	done

clean:
	@rm -rf $(BACKUPDIR)

push:
	@echo "Pushing to chezmoi..."
	chezmoi add $(CURDIR)

pull:
	@echo "Pulling from chezmoi..."
	chezmoi apply $(CURDIR)
