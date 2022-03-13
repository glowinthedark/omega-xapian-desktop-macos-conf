# MacOS Configuration of omega-xapian for desktop search

### Reference

- [Omega Quick Start](https://xapian.org/docs/omega/quickstart.html)
- [Xapian Downloads](https://xapian.org/download)

1. Install the omega-xapian binaries:

```bash
brew install omega
```

View the files installed by `brew`:

```bash
brew list omega

#/usr/local/Cellar/omega/1.4.18/bin/dbi2omega
#/usr/local/Cellar/omega/1.4.18/bin/htdig2omega
#/usr/local/Cellar/omega/1.4.18/bin/mbox2omega
#/usr/local/Cellar/omega/1.4.18/bin/omindex
#/usr/local/Cellar/omega/1.4.18/bin/omindex-list
#/usr/local/Cellar/omega/1.4.18/bin/scriptindex
#/usr/local/Cellar/omega/1.4.18/etc/omega.conf
#/usr/local/Cellar/omega/1.4.18/lib/xapian-omega/ (5 files)
#/usr/local/Cellar/omega/1.4.18/share/doc/ (9 files)
#/usr/local/Cellar/omega/1.4.18/share/man/ (3 files)
#/usr/local/Cellar/omega/1.4.18/share/omega/ (2 files)
```

Location of the CGI script for web search:

```
/usr/local/Cellar/omega/1.4.18/lib/xapian-omega/bin/omega
```

2. Download and extract the search templates from [xapian-omega-1.4.19.tar.xz](https://oligarchy.co.uk/xapian/1.4.19/xapian-omega-1.4.19.tar.xz) and copy the templates to `/var/lib/omega/templates`

```bash
cd /tmp
wget https://oligarchy.co.uk/xapian/1.4.19/xapian-omega-1.4.19.tar.xz 
tar xvf xapian-omega-1.4.19.tar.xz
mkdir -p /var/lib/omega
cp -r xapian-omega-1.4.19/templates /var/lib/omega/
```

3. Edit `omega.conf`

```bash
vi /usr/local/Cellar/omega/1.4.18/etc/omega.conf
```

Relevant keys:

```bash
# Directory containing Xapian databases:
database_dir /var/lib/omega

# Directory containing OmegaScript templates:
template_dir /var/lib/omega/templates
```

## Building the database index

> Check that `omindex` is in your `$PATH`: `which -a omindex`


```bash
omindex -v \
  --mime-type=WD3:ignore \
  --mime-type=json:ignore \
  --mime-type=db:ignore \
  --mime-type=py:text/plain \
  --ignore-exclusions \
  --db /var/lib/omega/default \
  --url /var/www \
  /Users/myuser/Documents/myfiles
  ```
