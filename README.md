## Protocol 1.1.0pre

### Subscriptions

    /Swarm#database!0.on+0user                  fresh client subscribes
    /Swarm#database!stamp+Yorigin.on+YuserSSN   client reconn (scoped on)
    /Swarm#database!now+Y.on+YuserNEW           client reon
    /Swarm#database!0+X.on+Y                    peer Y bootstraps from X
    /Swarm#database!bMarkTime+X.on+Y            peer Y reconnects to X
    /Swarm#database!bMarkTime+X.0+Y             log bookmark from X to Y
    /Object#stamp+XauthorSSN!0.on               client object sub
    /Object#stamp+XauthorSSN!versn+YauthrZZN.on client object re-sub
    /Object#stamp+XauthorSSN!pastTime.on        log segment request

### Ops

    /RPC#endpoint+Z!stamp+XauthrSSN.method+Z    RPC call
    /RPC#endpoint+Z!stamp+X1.stamp+XauthrSSN    RPC response
    /Object#id!stamp+YauthrSsn.field            LWW object write
    /Object+~o#id+author!stamp+orig.field       type parameters

### Messages

    /Message#up!stamp+Ysender.msg+0user         message to a user

### Source ids

    ~+X                                         upstream
    stamp+XuserSsn                              client incoming
    stamp                                       outgoing
    stamp+X                                     peer incoming
    stamp+~                                     pre-handshake
    0                                           internally generated

### Key-value db

    /Type#id!0-initOp+author.~
    /Type#id!stamp+origin.op
    /Type#id!stampOver-reorderd+orgn.op
    /Type#id!~-stamp.~
