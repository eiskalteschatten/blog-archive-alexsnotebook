Recently, I had the honor of setting up a Node.js-based web application on an Ubuntu server and I thought I would share the steps I took to get it up and running. The application runs on Node.js 22 and uses the [Fastify](https://fastify.dev/) web framework. The server runs Ubuntu 22.04 and I used Apache as a reverse proxy. These instructions should work with any Node-based web framework and any version of systemd-based Linux running Apache.

Deployment Process
------------------

Since the application is just a one-man, personal project, I wanted to keep the deployment process as simple as possible. I didn’t need any fancy workflows or CI, I just needed it to be simple and quick. The code is hosted in a private GitHub repository and I had already set up SSH keys for the server so that it can access my private repositories on GitHub. As such, my chosen method to update the code on the server wasn’t through a GitHub Action or other workflow, but rather via good ol’ fashioned git. I cloned the repository directly on the server and every time there is an update, I log in via SSH and pull the changes.

Of course, that isn’t the end of the story. I still have to install any new npm packages, build the application and have systemd restart the service that is in charge of keeping the application alive (more on that later). Since that requires multiple steps, I wrote a bash script to do it for me. It looks like this:

```bash
#!/bin/bash

cd /var/www/static-blog

echo "Pulling latest code from GitHub..."
git pull
wait

echo "Installing dependencies..."
sudo -u staticblog npm i
wait

echo "Building the project..."
sudo -u staticblog npm run build
wait

echo "Update file permissions..."
sudo chown -R staticblog:staticblog /var/www/static-blog

echo "Restarting the service..."
sudo systemctl restart staticblog
wait

echo "Deployment complete!"
```

There are a couple of things to note here. First of all, I run the Node application as its own user “staticblog”. Secondly, the SSH key I use to access the private repository was created with the user I log into the server with, which means any new changes pulled are automatically assigned to that user. That is why recursively reassigning the entire directory with the `chown -R` command is necessary.

Other than that, the deployment process is pretty straightforward.

Systemd
-------

As I mentioned above, I configured a systemd service to be responsible for keeping the application running as well as restarting the application after a reboot. That way, I don’t have to worry about it because the operating system will take care of all those things for me. As a bonus, logging is also automatically taken care of and accessible like with any other systemd-managed service.

I named the service configuration file `staticblog.service` and this is what it contains:

```bash
[Unit]
Description=Static Blog

[Service]
ExecStart=/var/www/static-blog/start.sh
Restart=always
User=staticblog
# Note Debian/Ubuntu uses 'nogroup', RHEL/Fedora uses 'nobody'
Group=nogroup
Environment=PATH=/usr/bin:/usr/local/bin
Environment=NODE_ENV=production
WorkingDirectory=/var/www/static-blog

[Install]
WantedBy=multi-user.target
```

To set up the service, I just made a symlink in the `/etc/systemd/system` folder to the staticblog.service file that I keep in the root of the repository. In this case, it would be to `/var/www/static-blog/staticblog.service`. After that, you just run `sudo systemctl daemon-reload` to reload the services and `sudo systemctl enable staticblog` to enable automatic restarting on reboot.

The following commands can be used to control the process:

```bash
# Start service
sudo systemctl start staticblog

# Restart service
sudo systemctl restart staticblog

# Follow logging
sudo journalctl --follow -u staticblog

# Enable restart on reboot
sudo systemctl enable staticblog

# Reload the systemd daemon
sudo systemctl daemon-reload
```

You may also have noticed that the service file uses the script `/var/www/static-blog/start.sh` to start the application which is a simple bash script that does nothing but run `npm start`. I used a bash script so that it would be easier to update with further steps in the future if necessary. This is what the script looks like:

```bash
#!/bin/bash

echo "Starting Static Blog..."

cd /var/www/static-blog
npm start

echo "Static Blog started"
```

Fortunately, it’s fairly easy to use systemd to manage the application and the extra stability is definitely worth the effort.

Apache Configuration
--------------------

The last piece of the puzzle is setting up the reverse proxy. In this case, I used Apache because the server already had it running for other, mostly PHP-based, applications. The general consensus on the internet is that nginx would have been a better choice to use as a reverse proxy, but I’m not going to install it when Apache is already running and does a perfectly good job too.

In any case, configuring the reverse proxy is really simple. I created a file named `staticblog.conf` that resides under `/etc/apache2/sites-available/`. This is what it looks like:

```bash
<VirtualHost *:80>
    ServerAdmin theadmins@emailaddress.com
    ServerName thestaticblogsdomain.com
    Protocols h2 http/1.1

    ProxyPreserveHost On
    ProxyPass / http://localhost:4000/
    ProxyPassReverse / http://localhost:4000/
</VirtualHost>
```

That’s all it takes. It’s simple and straightforward. The only thing to watch for is port 4000 which is what my Node application runs under. If yours is different, then you’ll have to update it accordingly.

After creating the configuration file, I just ran sudo `a2ensite staticblog.conf`, then `sudo systemctl reload apache2`. Then the website was enabled and accessible from the internet. I had already configured the DNS settings for the domain to point to my server’s IP address and, of course, also configured certificates for HTTPS, but those are both beyond the scope of this post.

Conclusion
----------

I have to admit that the main reason I wrote this up here is so that the next time I do it, I don’t have to search the internet for the various steps. Hopefully, it will also prove useful for others looking to do something similar. The process would pretty much be the same for any other language or framework.

If you use nginx or something other than systemd, the configuration would obviously be quite a bit different. Also, Linux distros not derived from Debian, such as RHEL, Fedora, openSUSE, etc, will have different paths for things such as the Apache configuration which is something to watch out for as well.

In any case, I hope this was helpful and if not, it will at least serve as great documentation for my future self if I ever end up setting up another Node application with this sort of server configuration.