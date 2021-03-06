# Kubernetes Client
[![Build Status](https://api.travis-ci.org/maclof/kubernetes-client.svg?branch=master)](https://travis-ci.org/maclof/kubernetes-client)

A PHP client for managing a Kubernetes cluster.

Last tested with 1.4.6 on Google Container Engine and 1.5.1 on Custom CoreOS Cluster.


## Installation using [Composer](http://getcomposer.org/)

```bash
$ composer require maclof/kubernetes-client
```

## Supported API Features
### v1
* Nodes
* Pods
* Replica Sets
* Replication Controllers
* Services
* Secrets
* Events
* Config Maps
* Endpoints

### extensions/v1beta1
* Deployments
* Jobs
* Ingresses


## Basic Usage

```php
<?php

require __DIR__ . '/vendor/autoload.php';

use Maclof\Kubernetes\Client;

$client = new Client([
	'master' => 'http://master.mycluster.com',
]);

// Find pods by label selector
$pods = $client->pods()->setLabelSelector([
	'name'    => 'test',
	'version' => 'a',
])->find();

// Find pods by field selector
$pods = $client->pods()->setFieldSelector([
	'metadata.name' => 'test',
])->find();

// Find first pod with label selector (same for field selector)
$pod = $client->pods()->setLabelSelector([
	'name' => 'test',
])->first();
```

## Authentication Examples

### Insecure HTTP
```php
$client = new Client([
	'master' => 'http://master.mycluster.com',
]);
```

### Using TLS Certificates (Client Certificate Validation)
```php
$client = new Client([
	'master'      => 'https://master.mycluster.com',
    'ca_cert'     => '/etc/kubernetes/ssl/ca.crt',
    'client_cert' => '/etc/kubernetes/ssl/client.crt',
    'client_key'  => '/etc/kubernetes/ssl/client.key',
]);
```

### Using Basic Auth
```php
$client = new Client([
	'master'   => 'https://master.mycluster.com',
    'username' => 'admin',
    'password' => 'abc123',
]);
```

### Using a Service Account
```php
$client = new Client([
	'master'  => 'https://master.mycluster.com',
	'ca_cert' => '/var/run/secrets/kubernetes.io/serviceaccount/ca.crt',
	'token'   => '/var/run/secrets/kubernetes.io/serviceaccount/token',
]);
```

## Usage Examples

### Create/Update a Replication Controller
```php
use Maclof\Kubernetes\Models\ReplicationController;

$replicationController = new ReplicationController([
	'metadata' => [
		'name' => 'nginx-test',
		'labels' => [
			'name' => 'nginx-test',
		],
	],
	'spec' => [
		'replicas' => 1,
		'template' => [
			'metadata' => [
				'labels' => [
					'name' => 'nginx-test',
				],
			],
			'spec' => [
				'containers' => [
					[
						'name'  => 'nginx',
						'image' => 'nginx',
						'ports' => [
							[
								'containerPort' => 80,
								'protocol'      => 'TCP',
							],
						],
					],
				],
			],
		],
	],
]);

if ($client->replicationControllers()->exists($replicationController->getMetadata('name'))) {
	$client->replicationControllers()->update($replicationController);
} else {
	$client->replicationControllers()->create($replicationController);
}
```

### Delete a Replication Controller
```php
$replicationController = $client->replicationControllers()->setLabelSelector([
	'name' => 'nginx-test',
])->first();
$client->replicationControllers()->delete($replicationController);
```
