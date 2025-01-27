There is an awesome script that does what I want, written by a legendary gentleman:
https://github.com/Szwendacz99/BookStack-Python-exporter

Please read his GitHub and now how i implemented that:
(this is only for linuxserver/bookstack image)

Bookstack backup:
On Alpine Linux, install Python:

apk add python3
apk add py3-setuptools
apk add py3-pip
download backup script and place it in the config folder:
GitHub

Clone the repo

next to the script place token.txt file containing token id and token secret in format: TOKEN_ID:TOKEN_SECRET

in the same directory run the command, specifying your app domain with https prefix (every parameter is optional as it have default value, this is an example):

python exporter.py
-H https://wiki.balgeriada.com
-f markdown
-l pages chapters books
--rate-limit 180
-c "/" "#"
--markdown-images
-t ./token.txt
-V debug
-p ./
--user-agent "Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/112.0"

Install cron:

apk add dcron
Create a script to run with cron:

nano exporter_script.sh
paste to script:

#!/bin/sh
python exporter.py \
    -H https://wiki.balgeriada.com \
    -f markdown \
    -l pages chapters books \
    --rate-limit 180 \
    -c "/" "#" \
    --markdown-images \
    -t ./token.txt \
    -V debug \
    -p ./ \
    --user-agent "Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/112.0"
Next, add the script to cron:

crontab -e
and paste:

0 2 * * * /config/backups/exporter_script.sh >> /config/backups/exporter.log 2>&1
0 2 * * * – This will run script on 2 AM

Run crone service:

s6-rc -u change svc-cron
Check if service is running:

s6-rc list svc-cron
To test cron:

crontab -e
and add:

* * * * * echo "Cron test" >> /tmp/cron_test.log 2>&1
Bonus --> Add discord notification:

Download the script from GitHub and place in the same directory as previous scripts.

Install curl:

apk add curl
modify exporter_script.sh script:

#!/bin/sh
python exporter.py \
    -H https://wiki.balgeriada.com \
    -f markdown \
    -l pages chapters books \
    --rate-limit 180 \
    -c "/" "#" \
    --markdown-images \
    -t ./token.txt \
    -V debug \
    -p ./ \
    --user-agent "Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/112.0"


current_datetime=$(date '+%Y-%m-%d %H:%M:%S')


./discord.sh --webhook-url="disco_webhook_here" --text "Bookstack backup: $current_datetime"
