email bridge

```
git clone https://github.com/JojiiOfficial/Matrix-EmailBridge
cd Matrix-EmailBridge/main
go get -v -u
go build -o emailbridge
./emailbridge
chmod 600 cfg.json data.db
mkdir ./temp
chmod -R 770 ./temp
```
`nano cfg.json`
