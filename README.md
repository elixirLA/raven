# libp2p Demos

## Demo 1:  DHT Peer & Content with Go and JS Nodes

**Directory**:  `content-dht-provide-find`

**What it demonstrates:**  A new DHT is created by the Go program `dht-interop`.  In a separate terminal or machine, a Node.js program connects to this DHT.  One connected, each verifies that it can find the other's content via the DHT.

**First terminal:**
```
cd content-dht-provide-find
make
./dht-interop -b ../util/private_key.bin.bootstrapper.Wa
```

`-b` means bootstrap mode.  In this example, the go program is always the bootstrap node, so `-b` is always required.

Note that the node ID of `dht-interop` is always `Qm...6aJ9oRuEzWa` because it is being read in from `../util/private_key.bin.bootstrapper.Wa` (a private key marshalled to X.509 generated by the program `util/private-key-gen`).  This is to keep the peer id of the bootstrap server stable across invocations.

**Second terminal:**  run the command printed out by dht-interop, replacing 127.0.0.1 with the IP of the server where dht-interop is listening.  Example:

Running the Node.js program:
```
cd content-dht-provide-find/js-dht-test
npm install  # first time only
node js-dht-test/index.js /ip4/127.0.0.1/tcp/5555/ipfs/QmehVYruznbyDZuHBV4vEHESpDevMoAovET6aJ9oRuEzWa
```



## Demo 2:  PubSub

**Directory**:  `pubsub`

**What it demonstrates**:  Two Go peers, one JS peer, and one Rust peer are all created and run a chat server using a shared PubSub topic.  Typing text in any peer sends it to all the other peers.

**Quick test**:  `cd pubsub` and then run `./test/test.sh`.  Requires Terminator (eg, `sudo apt-get install terminator`).  The rest of this section describes how to test manually.

(**TODO**:  eliminate centralized bootstrapper; any peer should be able to bootstrap from any other peer and peers should be able to start in any order)

**First terminal**:  Create the bootstrapper node

```
cd pubsub
./pubsub-interop -b ../util/private_key.bin.bootstrapper.Wa
```

The bootstrapper creates a new libp2p node, subscribes to the shared topic string, spawns a go routine to emit any publishes to that topic, and then waits forever.

**Second terminal**:  Create a go peer to connect to bootstrapper and publish on the topic

```
cd pubsub
./pubsub-interop ../util/private_key.bin.peer.Sk
```

This peer, which is not in bootstrapper mode, creates a node, subscribes to the shared topic string, spawns the same go routine, and then loops forever requesting user input and publishing each line to the topic.

**Third terminal**:  Create a JS peer to connect to bootstrap and publish on topic
```
cd pubsub/js
npm install  # first time only
node index.js /ip4/127.0.0.1/tcp/5555/ipfs/QmehVYruznbyDZuHBV4vEHESpDevMoAovET6aJ9oRuEzWa
```

This JS peer will fire off a hello message every few seconds, which the other two subscribing nodes can see.

**Fourth terminal**:  Createa a Rust peer to connect to the bootstrap node and then subscribe and publish on the topic:

```
cd pubsub/rust
cargo run
```

The Rust peer starts up, listens on port 6002, and then dials the boostrap peer.  (TODO:  rust-libp2p#471)  It is now subscribed to the same topic as the other peers.

If you return to the second, third or fourth terminals and type a message, the bootstrapper and the other 2 peers will all print your message.

In short, you have a chat app on a private libp2p network using PubSub.

## Debugging Notes

**JS** To see debug messages from the Node.js program, use the `DEBUG` environment variable:
```
DEBUG="libp2p:floodsub*,libp2p:switch*,mss:*" node index.js [args...]
```

**Go** To see debug messages in Go programs, do this at runtime:
```
IPFS_LOGGING=debug ./pubsub-interop [args...]
```

TODO:  describe custom instrumenting the local go code

If you instrument your go code with custom `fmt.Println`'s, then revert back like this:
```
cd $GOPATH
go get -u ./...
```

Other useful commands:
```
go get -u github.com/libp2p/go-libp2p-kad-dht   # fetch just Kad DHT repo
```


_Acknowledgements:  @jhiesey for DHT (content & peer routing) JS+Go interop, @stebalien for PubSub_
