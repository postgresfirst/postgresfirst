
> Linux-only, branded **meta-installer** for PostgreSQL with optional per-version packages.

PGF Postgres provides:

- A **single meta-package** `pgf-postgres` that lets you choose PostgreSQL **13 / 14 / 15 / 16 / 17** at install time.
- Optional **per-version kits** `pgf-postgres13|14|15|16|17` that default to a specific major.
- A multi-distro installer that uses the **official PGDG repos** by default (or the distro repos if you prefer).

---

## Features

- **Multi-distro** runtime: Debian/Ubuntu, RHEL/Rocky/Alma/Amazon Linux, openSUSE/SLES, Arch.
- **PostgreSQL majors:** 13–17.
- **Branded CLIs**
  - `pgf-postgres-setup` – installs & configures PostgreSQL.
  - `pgf-postgres` – start/stop/restart/status/logs (auto-detects the systemd unit).
  - `pgf-postgres-configure` – sets defaults in `/etc/pgf-postgres/config`.
  - Convenience wrappers: `pgf-postgres-setup-13|14|15|16|17`.
- **Safe by default:** binds to `localhost` unless you pass `--allow` (then `listen_addresses='*'` is set and pg_hba rules are appended).

---

## Repository layout

pgf-postgres/ # single meta-package (13–17 selection)
debian/ # Debian packaging (Debconf prompt)
rpm/ # RPM spec
installer/pgf-postgres-installer.sh
scripts/pgf-postgres*
pgf-postgres13/ # optional per-version kits (examples)
pgf-postgres14/
pgf-postgres15/
pgf-postgres16/
pgf-postgres17/
docs/assets/logo.png # logo used in README (add this file)

markdown
Copy
Edit

---

## Pre-requisites

- **Root/sudo** privileges and **systemd**.
- Outbound HTTPS for PGDG (unless using `--use-distro-repo`):
  - `https://apt.postgresql.org`, `https://download.postgresql.org`
- **Build tools** (for building .deb/.rpm from the kits):
  - Debian/Ubuntu: `build-essential debhelper devscripts debconf`
  - RHEL family: `rpm-build`
- **Resource guidance:** ≥10 GB free disk, ≥2 GB RAM (4 GB+ recommended).
- **Firewall/SELinux:** open port 5432 if needed; for non-default ports on SELinux:
  ```bash
  sudo semanage port -a -t postgresql_port_t -p tcp 5433
Build the meta-package
Debian/Ubuntu
bash
Copy
Edit
sudo apt-get update
sudo apt-get install -y build-essential debhelper devscripts debconf
cd pgf-postgres
debuild -us -uc
# Result: ../pgf-postgres_1.0.0-1_all.deb
RHEL/Rocky/Alma/Amazon Linux
bash
Copy
Edit
sudo dnf -y install rpm-build
cd pgf-postgres
tar czf /tmp/pgf-postgres-1.0.0.tar.gz --transform 's,^,pgf-postgres-1.0.0/,' installer scripts debian rpm LICENSE README.md
rpmbuild -ba rpm/pgf-postgres.spec --define "_sourcedir /tmp" --define "_version 1.0.0"
# Result: ~/rpmbuild/RPMS/noarch/pgf-postgres-1.0.0-1.noarch.rpm
To build per-version kits, run the same steps inside pgf-postgres13/ … pgf-postgres17/ and use the matching spec/control files there.

Install
Meta-package (pgf-postgres)
Debian/Ubuntu

bash
Copy
Edit
sudo apt-get install -y ./pgf-postgres_1.0.0-1_all.deb
# Debconf prompts for the default PG major (13–17).
# Change later with:
sudo dpkg-reconfigure pgf-postgres
RHEL family

bash
Copy
Edit
sudo dnf -y install ~/rpmbuild/RPMS/noarch/pgf-postgres-1.0.0-1.noarch.rpm
# Default major is 16; change with:
sudo pgf-postgres-configure 15
Per-version packages (pgf-postgres13|14|15|16|17)
Install the built artifact for your major (example: 15):

bash
Copy
Edit
# Debian/Ubuntu
sudo apt-get install -y ./pgf-postgres15_1.0.0-1_all.deb
# RHEL
sudo dnf -y install ~/rpmbuild/RPMS/noarch/pgf-postgres15-1.0.0-1.noarch.rpm
Per-version packages conflict with each other to avoid file overlap. Install only one at a time (or use the meta-package).

Provision PostgreSQL
Use defaults from /etc/pgf-postgres/config:

bash
Copy
Edit
sudo pgf-postgres-setup -y
Override on the fly:

bash
Copy
Edit
# Choose major, open VPC range, change port
sudo pgf-postgres-setup --version 14 --allow 10.0.0.0/8 --port 5433 -y

# Use distro repo instead of PGDG (offline/air-gapped)
sudo pgf-postgres-setup --use-distro-repo -y

# Custom data directory
sudo pgf-postgres-setup --data-dir /var/lib/postgresql/data-pgf -y
Convenience wrappers:

bash
Copy
Edit
sudo pgf-postgres-setup-13 -y
sudo pgf-postgres-setup-17 -y
Verify & operate
bash
Copy
Edit
pgf-postgres status
sudo -u postgres psql -c "select version();"
If you opened access (--allow), also open your firewall:

bash
Copy
Edit
# Ubuntu
sudo ufw allow 5432/tcp
# RHEL
sudo firewall-cmd --add-port=5432/tcp --permanent && sudo firewall-cmd --reload
Uninstall
Remove the PGF wrapper package (keeps data):

bash
Copy
Edit
# Debian
sudo apt-get remove pgf-postgres
# RHEL
sudo dnf remove -y pgf-postgres
Remove PostgreSQL software (via the installer):

bash
Copy
Edit
sudo pgf-postgres-setup --uninstall -y
# And purge data directories (irreversible):
sudo pgf-postgres-setup --uninstall --purge -y
Troubleshooting
Debian/Ubuntu PGDG key/codename

Key installs to /etc/apt/keyrings/postgresql.gpg.

Ensure /etc/apt/sources.list.d/pgdg.list uses your correct codename (e.g., jammy-pgdg), then sudo apt-get update.

RHEL AppStream conflict

bash
Copy
Edit
sudo dnf -qy module disable postgresql
sudo pgf-postgres-setup -y
Amazon Linux

Use --use-distro-repo or preinstall the matching PGDG repo package.

SUSE/SLES

If the public zypper repo isn’t available for your exact version, use --use-distro-repo.

Service unit detection

bash
Copy
Edit
systemctl list-unit-files | grep -i postgres
If multiple majors are installed, the helper picks the first matching unit; adjust as needed.

FAQ
What is a meta-package?
A package that mostly carries dependencies/scripts; it doesn’t include PostgreSQL itself. Installing it pulls in (or installs via scripts) the chosen stack.

Can I run multiple PG majors side-by-side?
PGDG supports parallel installs (versioned units like postgresql-15, postgresql-16). The pgf-postgres helper targets one active unit by default; if you need multiple clusters concurrently, manage additional units manually.

Windows support?
Intentionally not supported.

Contributing
Open PRs with clear descriptions.

Keep the installer idempotent and safe for re-runs.

For Debian, update debian/changelog; for RPM, bump _version at build time.

License
MIT. See LICENSE.

Maintainer notes
Brand assets: place your logo at docs/assets/logo.png and update the README path if needed.

To publish release artifacts, attach built .deb/.rpm files to a GitHub Release and include SHA256 checksums.

