https://gitlab.com/robintown/mx-puppet-groupme

git clone https://gitlab.com/robintown/mx-puppet-groupme

cd mx-puppet-groupme

npm install groupme

cp sample.config.yaml config.yaml

nano config.yaml

npm run start -- --register

`/usr/local/bin/node /root/mx-puppet-groupme/build/index.js --config=/root/mx-puppet-groupme/config.yaml --registration-file=/root/mx-puppet-groupme/groupme-registration.yaml`
