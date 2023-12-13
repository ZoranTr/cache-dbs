# cache-dbs
_Benchmarking ready tool for nosql databases: **redis, dragonflydb and keydb**_

We will try to do some basic benchmarks on one of the best nosql services. They all have some benchmarks provided on the official documentation but of course, they may use different benchmarking tools with different hardware technology and etc.
Because of that we will try to do benchmark by our self.

First lets prepare the docker containers for benchmarking the services, they will all have the same config with same memory resources.
If you still have not installed docker please try this out: https://docs.docker.com/engine/install/.

Once it is installed and you have cloned this repository run:

`docker-compose up -d`

I had a lot of troubles with the memory required for the dragonflydb


_maxmemory has not been specified. Deciding myself...._

_Found 1.04GiB available memory. Setting maxmemory to 850.97MiB_


so I updated the docker resources as: CPUs: 4	Memory: 3.50 GB

Than after running `docker exec -it cache-dbs-dev-app sh`, you may run the [memtier_benchmark](https://github.com/RedisLabs/memtier_benchmark) command used for most of the nosql services benchmark tests
Before any benchmark tests I checked the `docker stats`. Note that you may get different stats based on the resources.

| NAME                     | CPU %    | MEM USAGE / LIMIT    | MEM %   | NET I/O         | BLOCK I/O      | 
|--------------------------|----------|----------------------|---------|-----------------|----------------|
| cache-dbs-dev-app        | 0.03%    | 12.54MiB / 3.35GiB   | 0.37%   | 366MB / 460MB   | 0B / 26.4MB    |
| cache-dbs-dev-redis      | 0.49%    | 2.699MiB / 128MiB    | 2.11%   | 293MB / 199MB   | 0B / 168kB     |
| cache-dbs-dev-dragonfly  | 11.58%   | 55.94MiB / 128MiB    | 43.70%  | 83.6MB / 90MB   | 0B / 0B        |
| cache-dbs-dev-keydb      | 1.58%    | 5.789MiB / 128MiB    | 4.52%   | 83.6MB / 75.3MB | 94.2kB / 111kB |
| cache-dbs-dev-nginx      | 0.00%    | 3.832MiB / 3.35GiB   | 0.11%   | 2.61kB / 0B     | 0B / 0B        |

So from the stats it is obvious that dragonflydb uses a lot of memory, as it is written in the official documentation that the best performance comes out after 4GB of memory :D 

You may as well try the build-in benchmark programs for redis and dragonfly (keydb doesnt have it inbuild):

`docker exec -it cache-dbs-dev-app sh` and from there run `redis-benchmark -q` :

| COMMAND        | dragonflydb requests/s | redis requests/s     |
|----------------|------------------|----------------------|
| PING_INLINE    | 6300.40          | 7809.45              |
| PING_BULK      | 6162.95          | 8533.88 (PING_MBULK) |
| SET            | 4524.27          | 8688.10              |
| GET            | 4512.84          | 9009.82              |
| INCR           | 4572.26          | 8164.60              |
| LPUSH          | 4220.48          | 7818.00              |
| RPUSH          | 4469.87          | 7953.55              |
| LPOP           | 4535.56          | 7574.04              |
| RPOP           | 4638.86          | 7393.72              |
| SADD           | 4527.55          | 8491.13              |
| HSET           | 4472.87          | 8190.68              |
| SPOP           | 4680.11          | 7848.68              |
| LPUSH          | 4348.96          | 7481.67              |
| LRANGE_100     | 4295.90          | 5101.00              |
| LRANGE_300     | 2854.04          | 3068.71              |
| LRANGE_500     | 2734.41          | 2333.29              |
| LRANGE_600     | 2110.51          | 2158.15              |
| MSET (10 keys) | 3794.06          | 7202.54              |

Redis showed much better results in those benchmarks. But after checking out the memtier_benchmark results, you may be surprised especially as the threads increase.

`memtier_benchmark -s cache-dbs-dev-redis.cache-dbs_cache-dbs-app-network -p 6379 --ratio 1:0 -c 20 --test-time 100 -t 4 -d 256 --distinct-client-seed`
just change the options to compare different results on different servers, full documentation for the options you may find here [memtier_benchmark](https://github.com/RedisLabs/memtier_benchmark) 
there is nginx ready container for localhost in case you want to try something from the application level with composer.

Have fun!