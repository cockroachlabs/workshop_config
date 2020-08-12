## Docker and Brew install on OSX

```
brew install cockroachdb/tap/cockroach
docker pull cockroachdb/cockroach:v20.1.4
```

## Manual Install OSX Example

Demo of quick install of test clusters.  First get the binary and extract the binary.
```
wget https://binaries.cockroachdb.com/cockroach-v20.1.4.darwin-10.9-amd64.tgz

tar xvf cockroach-v20.1.4.darwin-10.9-amd64.tgz
x cockroach-v20.1.4.darwin-10.9-amd64/cockroach

cd ./cockroach-v20.1.4.darwin-10.9-amd64
```

### Single Node Startup

```
./cockroach start-single-node --insecure

*
* WARNING: RUNNING IN INSECURE MODE!
*
* - Your cluster is open for any client that can access <all your IP addresses>.
* - Any user, even root, can log in without providing a password.
* - Any user, connecting as root, can read or write any data in your cluster.
* - There is no network encryption nor authentication, and thus no confidentiality.
*
* Check out how to secure your cluster: https://www.cockroachlabs.com/docs/v20.1/secure-a-cluster.html
*
*
* WARNING: neither --listen-addr nor --advertise-addr was specified.
* The server will advertise "glennfawcett.local" to other nodes, is this routable?
*
* Consider using:
* - for local-only servers:  --listen-addr=localhost
* - for multi-node clusters: --advertise-addr=<host/IP addr>
*
*
*
* INFO: Replication was disabled for this cluster.
* When/if adding nodes in the future, update zone configurations to increase the replication factor.
*
CockroachDB node starting at 2020-08-12 21:56:55.021002 +0000 UTC (took 0.2s)
build:               CCL v20.1.4 @ 2020/07/29 22:51:52 (go1.13.9)
webui:               http://glennfawcett.local:8080
sql:                 postgresql://root@glennfawcett.local:26257?sslmode=disable
RPC client flags:    ./cockroach <client cmd> --host=glennfawcett.local:26257 --insecure
logs:                /tmp/crdb/cockroach-v20.1.4.darwin-10.9-amd64/cockroach-data/logs
temp dir:            /tmp/crdb/cockroach-v20.1.4.darwin-10.9-amd64/cockroach-data/cockroach-temp510957049
external I/O path:   /tmp/crdb/cockroach-v20.1.4.darwin-10.9-amd64/cockroach-data/extern
store[0]:            path=/tmp/crdb/cockroach-v20.1.4.darwin-10.9-amd64/cockroach-data
storage engine:      rocksdb
status:              initialized new cluster
clusterID:           b80dc2e5-076e-4e82-afe7-7e3431bf430a
nodeID:              1
```

### Single Line Demo Startup
```
./cockroach demo --geo-partitioned-replicas

#
# --geo-partitioned replicas operates on a 9 node cluster.
# The cluster size has been changed from the default to 9 nodes.
#
# Welcome to the CockroachDB demo database!
#
# You are connected to a temporary, in-memory CockroachDB cluster of 9 nodes.
#
# This demo session will attempt to enable enterprise features
# by acquiring a temporary license from Cockroach Labs in the background.
# To disable this behavior, set the environment variable
# COCKROACH_SKIP_ENABLING_DIAGNOSTIC_REPORTING=true.
#
# Beginning initialization of the movr dataset, please wait...
#
# Waiting for license acquisition to complete...
#
# Partitioning the demo database, please wait...
#
# The cluster has been preloaded with the "movr" dataset
# (MovR is a fictional vehicle sharing company).
#
# Reminder: your changes to data stored in the demo session will not be saved!
#
# Connection parameters:
#   (console) http://127.0.0.1:58434
#   (sql)     postgres://root:admin@?host=%2Fvar%2Ffolders%2F7k%2F80ybhw09733cf4_wptydwm4c0000gn%2FT%2Fdemo666803698&port=26257
#   (sql/tcp) postgres://root@127.0.0.1:58436?sslmode=disable
#
# To display connection parameters for other nodes, use \demo ls.
#
# Cockroach demo is running in insecure mode.
# Run with --insecure=false to use security related features.
# Note: Starting in secure mode will become the default in v20.2.
#
# Server version: CockroachDB CCL v20.1.4 (x86_64-apple-darwin14, built 2020/07/29 22:51:52, go1.13.9) (same version as client)
# Cluster ID: c9ec1354-5b34-4ba3-8e9e-3420c90e38f5
# Organization: Cockroach Demo
#
# Enter \? for a brief introduction.
#
root@127.0.0.1:58436/movr> show locality;
        locality
------------------------
  region=us-east1,az=b
(1 row)

Time: 449µs


root@127.0.0.1:58436/movr> show create table rides;
  table_name |                                                        create_statement
-------------+----------------------------------------------------------------------------------------------------------------------------------
  rides      | CREATE TABLE rides (
             |     id UUID NOT NULL,
             |     city VARCHAR NOT NULL,
             |     vehicle_city VARCHAR NULL,
             |     rider_id UUID NULL,
             |     vehicle_id UUID NULL,
             |     start_address VARCHAR NULL,
             |     end_address VARCHAR NULL,
             |     start_time TIMESTAMP NULL,
             |     end_time TIMESTAMP NULL,
             |     revenue DECIMAL(10,2) NULL,
             |     CONSTRAINT "primary" PRIMARY KEY (city ASC, id ASC),
             |     CONSTRAINT fk_city_ref_users FOREIGN KEY (city, rider_id) REFERENCES users(city, id),
             |     CONSTRAINT fk_vehicle_city_ref_vehicles FOREIGN KEY (vehicle_city, vehicle_id) REFERENCES vehicles(city, id),
             |     INDEX rides_auto_index_fk_city_ref_users (city ASC, rider_id ASC) PARTITION BY LIST (city) (
             |         PARTITION us_west VALUES IN (('seattle'), ('san francisco'), ('los angeles')),
             |         PARTITION us_east VALUES IN (('new york'), ('boston'), ('washington dc')),
             |         PARTITION europe_west VALUES IN (('amsterdam'), ('paris'), ('rome'))
             |     ),
             |     INDEX rides_auto_index_fk_vehicle_city_ref_vehicles (vehicle_city ASC, vehicle_id ASC) PARTITION BY LIST (vehicle_city) (
             |         PARTITION us_west VALUES IN (('seattle'), ('san francisco'), ('los angeles')),
             |         PARTITION us_east VALUES IN (('new york'), ('boston'), ('washington dc')),
             |         PARTITION europe_west VALUES IN (('amsterdam'), ('paris'), ('rome'))
             |     ),
             |     FAMILY "primary" (id, city, vehicle_city, rider_id, vehicle_id, start_address, end_address, start_time, end_time, revenue),
             |     CONSTRAINT check_vehicle_city_city CHECK (vehicle_city = city)
             | ) PARTITION BY LIST (city) (
             |     PARTITION us_west VALUES IN (('seattle'), ('san francisco'), ('los angeles')),
             |     PARTITION us_east VALUES IN (('new york'), ('boston'), ('washington dc')),
             |     PARTITION europe_west VALUES IN (('amsterdam'), ('paris'), ('rome'))
             | );
             | ALTER PARTITION europe_west OF INDEX movr.public.rides@primary CONFIGURE ZONE USING
             |     constraints = '[+region=europe-west1]';
             | ALTER PARTITION us_east OF INDEX movr.public.rides@primary CONFIGURE ZONE USING
             |     constraints = '[+region=us-east1]';
             | ALTER PARTITION us_west OF INDEX movr.public.rides@primary CONFIGURE ZONE USING
             |     constraints = '[+region=us-west1]';
             | ALTER PARTITION europe_west OF INDEX movr.public.rides@rides_auto_index_fk_city_ref_users CONFIGURE ZONE USING
             |     constraints = '[+region=europe-west1]';
             | ALTER PARTITION us_east OF INDEX movr.public.rides@rides_auto_index_fk_city_ref_users CONFIGURE ZONE USING
             |     constraints = '[+region=us-east1]';
             | ALTER PARTITION us_west OF INDEX movr.public.rides@rides_auto_index_fk_city_ref_users CONFIGURE ZONE USING
             |     constraints = '[+region=us-west1]';
             | ALTER PARTITION europe_west OF INDEX movr.public.rides@rides_auto_index_fk_vehicle_city_ref_vehicles CONFIGURE ZONE USING
             |     constraints = '[+region=europe-west1]';
             | ALTER PARTITION us_east OF INDEX movr.public.rides@rides_auto_index_fk_vehicle_city_ref_vehicles CONFIGURE ZONE USING
             |     constraints = '[+region=us-east1]';
             | ALTER PARTITION us_west OF INDEX movr.public.rides@rides_auto_index_fk_vehicle_city_ref_vehicles CONFIGURE ZONE USING
             |     constraints = '[+region=us-west1]'
(1 row)

Time: 192.22ms
```


### Manual Install 3 nodes on Laptop
```
./cockroach start --insecure --store=node1 --listen-addr=localhost:26257 --http-addr=localhost:8081 --join=localhost:26257,localhost:26258,localhost:26259 --background
./cockroach start --insecure --store=node2 --listen-addr=localhost:26258 --http-addr=localhost:8082 --join=localhost:26257,localhost:26258,localhost:26259 --background
./cockroach start --insecure --store=node3 --listen-addr=localhost:26259 --http-addr=localhost:8083 --join=localhost:26257,localhost:26258,localhost:26259 --background

./cockroach init --insecure --host=localhost:26257
```