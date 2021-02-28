# curl github public key
```bash
# variable for username
UNAME='substack'

# see all public keys for user and grep specific keys
curl -s https://github.com/$UNAME.keys | grep ^ssh-ed25519
```

#unix #bash #tip 