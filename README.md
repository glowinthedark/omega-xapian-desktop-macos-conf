# MacOS Configuration of omega-xapian search

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

#/usr/local/Cellar/omega/1.4.21/bin/dbi2omega
#/usr/local/Cellar/omega/1.4.21/bin/htdig2omega
#/usr/local/Cellar/omega/1.4.21/bin/mbox2omega
#/usr/local/Cellar/omega/1.4.21/bin/omindex
#/usr/local/Cellar/omega/1.4.21/bin/omindex-list
#/usr/local/Cellar/omega/1.4.21/bin/scriptindex
#/usr/local/Cellar/omega/1.4.21/etc/omega.conf
#/usr/local/Cellar/omega/1.4.21/lib/xapian-omega/ (5 files)
#/usr/local/Cellar/omega/1.4.21/share/doc/ (9 files)
#/usr/local/Cellar/omega/1.4.21/share/man/ (3 files)
#/usr/local/Cellar/omega/1.4.21/share/omega/ (2 files)
```

Location of the CGI script for web search:

```
/usr/local/Cellar/omega/1.4.21/lib/xapian-omega/bin/omega
```

2. Download and extract the search templates from [xapian-omega-1.4.21.tar.xz](https://oligarchy.co.uk/xapian/1.4.21/xapian-omega-1.4.21.tar.xz) (check the [latest version](https://oligarchy.co.uk/xapian/)) and copy the `templates` folder to `/var/lib/omega/templates`

```bash
cd /tmp
wget https://oligarchy.co.uk/xapian/1.4.21/xapian-omega-1.4.21.tar.xz 
tar xvf xapian-omega-1.4.21.tar.xz
sudo mkdir -p /var/lib/omega
sudo chown `whoami` /var/lib/omega
cp -r xapian-omega-1.4.21/templates /var/lib/omega/

## after the copy operation your /var/lib/omega/templates should look similar to this:
# $ tree /var/lib/omega/templates
# /var/lib/omega/templates
# ├── godmode
# ├── inc
# │   ├── anyalldropbox
# │   ├── anyallradio
# │   └── toptermsjs
# ├── opensearch
# ├── query
# ├── topterms
# └── xml
```

3. Edit `omega.conf`

```bash
vi /usr/local/Cellar/omega/1.4.21/etc/omega.conf
```

Relevant keys:

```bash
# Directory containing Xapian databases:
database_dir /var/lib/omega/data

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
  
  Useful flags that could be useful in order to configure and include/exclude file types:
  
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

### Python dev server

Assuming you used `/var/www` as the web root when you ran `omindex` with e.g. `--url /var/www`:

```bash
mkdir -p /var/www/cgi-bin

# copy the `omega` CGI script to the `cgi-bin` folder under the web root:
cp /usr/local/Cellar/omega/1.4.18/lib/xapian-omega/bin/omega /var/www/cgi-bin/

# start the python dev server from the web root folder:
cd /var/www
python3 -m http.server --cgi 8000 --directory .
```

If everything went well then the omega search page should now be accessible at:

- http://localhost:8000/cgi-bin/omega


### Caddy server with fcgiwrap

1. Intall `fcgiwrap`:

```bash
brew install fcgiwrap
```

2. Assuming `/var/www` is your web root `cd` into it, create a `cgi-bin` subfolder and launch `fcgiwrap` to run in background:

```
cd /var/www
mkdir -p cgi-bin
fcgiwrap -f -s tcp:127.0.0.1:8999 &
```
3. Create or update the `/etc/caddy/Caddyfile` to include the reverse proxy for `fcgiwrap`:

`sudo vi /etc/caddy/Caddyfile`

```sh
http:// {
	root * /var/www

  # optional: enable the file-server browser module to enable directory listsings
	file_server browse {
		root /var/www
	}	

	reverse_proxy /cgi-bin/* localhost:8999 {
		transport fastcgi {
			split .pl
		}
	}
}
```

4. Start `caddy`:

```bash
caddy start --config /etc/caddy/Caddyfile
```

To activate and start `caddy` and `fcgiwrap` using the included plist files [caddy.plist](caddy_launcher.plist) and [fcgiwrap.plist](fcgiwrap_launcher.plist):

```bash
launchctl load -w ~/Library/LaunchAgents/fcgiwrap.plist
launchctl load -w ~/Library/LaunchAgents/caddy.plist
```

Stop caddy via launchd:
```bash
launchctl unload -w ~/Library/LaunchAgents/caddy.plist
launchctl unload -w ~/Library/LaunchAgents/fcgiwrap.plist
```

### Caddy server with native CGI plugin

You can also run caddy with the the native 3rd party CGI plugin: 

1. Go to https://caddyserver.com/download and enable the `http.handlers.cgi` plugin.
2. Click the **Download** button.
3. Use a `Caddyfile` like the one below:




The search page should now be available at http://localhost/cgi-bin/omega (or on whatever other port that you have configured `caddy` to use).

```bash
{
	# debug
	# http_port 8080
	order cgi last
	order file_server last

	log {
		output stderr
		format console
		level DEBUG
	}
}

http:// {
	root * /

	file_server browse {
		root /docs/my-files-are-here
	}

	## this is the 'omega' file copied from /usr/local/Cellar/omega/1.4.18/lib/xapian-omega/bin/omega
	cgi /omega* /var/www/cgi-bin/omega {
		script_name /omega
		dir /var/www/cgi-bin
		pass_all_env
	}
}
```

A custom `caddy` build can also be generated from the command line with:

```bash
# 1. install the xcaddy build tool: 
go install github.com/caddyserver/xcaddy/cmd/xcaddy@latest

# 2. make the build with the custom plugin:
xcaddy build --with github.com/aksdb/caddy-cgi/v2
```
