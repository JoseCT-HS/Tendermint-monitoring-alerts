# Tendermint monitoring alerts for Agoric
Instructions to have in Telegram a monitoring tool for Tendermint chains. This tutorial has been created for the Agoric project and its community. You can get more information about the project in the following links:
Official website: https://agoric.com/
Discord Channel: https://discord.com/invite/qDW8DRes4s
Telegram Group: https://t.me/agoricsystems
Medium: https://medium.com/agoric
YouTube Channel: https://www.youtube.com/channel/UCpY91oQLh_Lp0mitdZ5bYWg
Github: https://github.com/agoric

## Instructions
There is a monitoring software that allows to notify via Telegram programs with Tendermint strings. This software is called PANIC. To configure it we must execute the following commands:
```
sudo apt-get install redis-server  -y
sudo systemctl enable redis-server.service  # set Redis to start on boot
sudo nano /etc/redis/redis.conf
```

Then, we will have to edit the redis.conf file as follow:
- In the GENERAL section add the line `bind 127.0.0.1 ::1`. 
- In the SECURITY section add the line `requirepass <PASS>`, where <PASS> will be the password of the service.
  - Uncomment, or add, the next lines in case they don't exist.
  ```
  rename-command FLUSHALL ""
  rename-command PEXPIRE ""
  rename-command CONFIG ""
  rename-command SHUTDOWN ""
  rename-command BGREWRITEAOF ""
  rename-command BGSAVE ""
  rename-command SAVE ""
  rename-command SPOP ""
  rename-command SREM ""
  rename-command RENAME ""
  rename-command DEBUG ""
  ```
Once this is done, we can restart the REDIS service.
```
sudo service redis-server restart
sudo service redis-server status
```

The next step will be to install the PANIC packages. We will have into account that this tool is only compatible with Python 3.6 or higher, so depending on the alias we have in the system we will have to use Python or Phyton3. We can check the version we have installed using the command `python -V`.
```
sudo apt-get install python3-pip -y
sudo pip3 install pipenv 
```

Now, it is recommended to creaate a new user where we are going to run the PANIC tool.
```
sudo adduser panic_alerter
sudo mkdir /opt/panic_alerter
sudo chown -R panic_alerter:panic_alerter /opt/panic_alerter
su panic_alerter
cd
cd /opt/panic_alerter/ && git clone https://github.com/SimplyVC/panic_cosmos.git && cd panic_cosmos 
pipenv sync
pipenv run python run_setup.py
```
At this point, we have almost finished with the configuration. The next thing to do would be to choose where do we want to receive those alerts. In this tutorial we will be using a Telegram Bot, so we will need its API.

Finally, we will create the service. 

```
sudo tee <<EOF >/dev/null /etc/systemd/system/panic_alerter.service
[Unit]
Description=P.A.N.I.C.
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
User=panic_alerter
TimeoutStopSec=90s
WorkingDirectory=/opt/panic_alerter/panic_cosmos
ExecStart=/usr/local/bin/pipenv run python /opt/panic_alerter/panic_cosmos/run_alerter.py

[Install]
WantedBy=multi-user.target
EOF
```

You will need to enter the following commands to enable and start the service that has been previously created
```
sudo systemctl enable panic_alerter
sudo systemctl daemon-reload
sudo systemctl start panic_alerter
sudo systemctl status panic_alerter
```
