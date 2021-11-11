# [PGO](https://github.com/CrunchyData/postgres-operator), Crunchy [Postgres Operator](https://github.com/CrunchyData/postgres-operator) Examples

## Install PGO (https://github.com/CrunchyData/postgres-operator/releases)

```
$ wget https://github.com/CrunchyData/postgres-operator/releases/download/v4.7.3/pgo

$ sudo mv pgo /usr/local/bin/pgo

$ sudo chmod +x /usr/local/bin/pgo

Audit: pgo --help
```

## Install PGO, the Postgres Operator
```
kubectl apply -k kustomize/install

*** This will create a namespace called postgres-operator and create all of the objects required to deploy PGO.

Audit:
kubectl -n postgres-operator get pods \
  --selector=postgres-operator.crunchydata.com/control-plane=postgres-operator \
  --field-selector=status.phase=Running

NAME                                READY   STATUS    RESTARTS   AGE
postgres-operator-9dd545d64-t4h8d   1/1     Running   0          3s
```

## Create a Postgres Cluster
```
kubectl apply -k kustomize/postgres

Audit:

kubectl -n postgres-operator describe postgresclusters.postgres-operator.crunchydata.com hippo
```

## Connect 
```
## Directly

If you are on the same network as your PostgreSQL cluster, you can connect directly to it using the following command
** From master or worker node:

psql $(kubectl -n postgres-operator get secrets hippo-pguser-hippo -o go-template='{{.data.uri | base64decode}}')

## Using a Port-Forward

In a new terminal, create a port forward:

PG_CLUSTER_PRIMARY_POD=$(kubectl get pod -n postgres-operator -o name \
  -l postgres-operator.crunchydata.com/cluster=hippo,postgres-operator.crunchydata.com/role=master)
kubectl -n postgres-operator port-forward "${PG_CLUSTER_PRIMARY_POD}" 5432:5432

## Establish a connection to the PostgreSQL cluster.

PG_CLUSTER_USER_SECRET_NAME=hippo-pguser-hippo

PGPASSWORD=$(kubectl get secrets -n postgres-operator "${PG_CLUSTER_USER_SECRET_NAME}" -o go-template='{{.data.password | base64decode}}') \
PGUSER=$(kubectl get secrets -n postgres-operator "${PG_CLUSTER_USER_SECRET_NAME}" -o go-template='{{.data.user | base64decode}}') \
PGDATABASE=$(kubectl get secrets -n postgres-operator "${PG_CLUSTER_USER_SECRET_NAME}" -o go-template='{{.data.dbname | base64decode}}') \
psql -h localhost
```

### Additional
```
# Client TLS 
First, create a directory to hold these files by running the following command:

mkdir ${HOME?}/.pgo
chmod 700 ${HOME?}/.pgo

# Next, copy the certificates to this new directory:

cp /PATH/TO/client.crt ${HOME?}/.pgo/client.crt && chmod 600 ${HOME?}/.pgo/client.crt
cp /PATH/TO/client.pem ${HOME?}/.pgo/client.pem && chmod 400 ${HOME?}/.pgo/client.pem

# Set the following environment variables to point to the client TLS files:

cat <<EOF >> ${HOME?}/.bashrc
export PGO_CA_CERT="${HOME?}/.pgo/client.crt"
export PGO_CLIENT_CERT="${HOME?}/.pgo/client.crt"
export PGO_CLIENT_KEY="${HOME?}/.pgo/client.pem"
EOF

# Apply those changes to the current session by running:

source ~/.bashrc

# Configuring pgouser
The pgouser file contains the username and password used for authentication with the Crunchy PostgreSQL Operator.

To setup the pgouser file, run the following:

echo "<USERNAME_HERE>:<PASSWORD_HERE>" > ${HOME?}/.pgo/pgouser
cat <<EOF >> ${HOME?}/.bashrc
export PGOUSER="${HOME?}/.pgo/pgouser"
EOF

# Apply those changes to the current session by running:

source ${HOME?}/.bashrc

# Configuring the API Server URL
If the Crunchy PostgreSQL Operator is not accessible outside of the cluster, it’s required to setup a port-forward tunnel using the kubectl or oc binary.

In a separate terminal we need to setup a port forward to the Crunchy PostgreSQL Operator to ensure connection can be made outside of the cluster:

kubectl port-forward -n pgo svc/postgres-operator 8443:8443

Note: The port-forward will be required for the duration of using the PostgreSQL client.

Next, set the following environment variable to configure the API server address:

cat <<EOF >> ${HOME?}/.bashrc
export PGO_APISERVER_URL="https://<IP_OF_OPERATOR_API>:8443"
EOF

Note: if port-forward is being used, the IP of the Operator API is 127.0.0.1

Apply those changes to the current session by running:

source ${HOME?}/.bashrc
```
