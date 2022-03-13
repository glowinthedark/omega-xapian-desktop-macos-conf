# MacOS Configuration of omega-xapian for desktop search

## Reference links

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

First check that `omindex` is in your `$PATH`: `which -a omindex` and then run:

```bash
omindex -v \
  --mime-type=json:ignore \
  --mime-type=xml:ignore \
  --mime-type=py:text/plain \
  --ignore-exclusions \
  --db /var/lib/omega/default \
  --url /var/www \
  /Users/myuser/Documents/myfiles
  ```
  
  Useful flags that be use to include/exclude file types:
  
  ```
  
# -M, --mime-type=EXT:TYPE  assume any file with extension EXT has MIME
#                           Content-Type TYPE, instead of using libmagic
#                           (empty TYPE removes any existing mapping for EXT;
#                           other special TYPE values: 'ignore' and 'skip')

# -G, --mime-type-match=GLOB:TYPE
#                           assume any file with leaf name matching shell
#                           wildcard pattern GLOB has MIME Content-Type TYPE
#                           (special TYPE values: 'ignore' and 'skip')
```

> See all command line flags with `omindex --help`

## Running the server

#### Python dev server

Switch to the directory you configured as web root when you ran `omindex` with `--url /var/www`:

```bash
mkdir -p /var/www/cgi-bin

# copy the CGI script to the web server's `cgi-bin` folder
cp /usr/local/Cellar/omega/1.4.18/lib/xapian-omega/bin/omega /var/www/cgi-bin/

# start the python dev server
cd /var/www
python3 -m http.server --cgi 8000 --directory .
```

If everything went well then the omega search page should now be accessible at:

- http://localhost:8000/cgi-bin/omega
