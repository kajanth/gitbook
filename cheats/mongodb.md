# MongoDB

## Clients

### K8s port forward

Required role.

```bash
...
rules:
- apiGroups: [""]
  resources: ["pods/portforward"]
  verbs: ["get", "list", "create"]
...
```

Create the port forwarding.

```bash
kubectl -n YOUR-MONGO-NS port-forward \
  --address 0.0.0.0
  service/YOUR-MONGO-SERVICE-NAME 27017:27017
```

Then connect to mongo.

```bash
mongo -host 127.0.0.1 --port 27017 -u root --password XXXXXX
```

### Ops Manager

{% embed url="https://www.mongodb.com/products/ops-manager" %}



## Backup

### Create backup

```bash
mongodump --db your-db-name
```

### Restore backup

```bash
mongorestore --db your-db-name dump/your-db-name
```

## Collections management

### List collections by size

```bash
use YOUR-DATABASE

var collectionNames = db.getCollectionNames(), stats = [];
collectionNames.forEach(function (n) { stats.push(db[n].stats()); });
stats = stats.sort(function(a, b) { return b['size'] - a['size']; });
for (var c in stats) { print(stats[c]['ns'] + ": " + stats[c]['size'] + " (" + stats[c]['storageSize'] + ")"); }
```

## User management

### Add user

```bash
db.addUser({user: "development", pwd: "123456", roles: [ "dbAdmin"]});
```

Or:

```bash
db.createUser({user: "development", pwd: "123456", roles: [ "dbAdmin"]});
```



