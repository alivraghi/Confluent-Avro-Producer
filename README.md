# NAME

Kafka::Producer::Avro - Avro message producer for Apache Kafka.

# SYNOPSIS

    use Kafka::Connection;
    use Kafka::Producer::Avro;

    my $connection = Kafka::Connection->new( host => 'localhost' );

    my $producer = Kafka::Producer::Avro->new( Connection => $connection );

    # Do some interactions with Avro & SchemaRegistry before sending messages

    # Sending a single message
    my $response = $producer->send(...);

    # Sending a series of messages
    $response = $producer->send(...);

    # Closes the producer and cleans up
    undef $producer;
    $connection->close;
    undef $connection;

# DESCRIPTION

`Kafka::Producer::Avro` main feature is to provide object-oriented API to 
produce messages according to _Confluent SchemaRegistry_ and _Avro_ serialization.

`Kafka::Producer::Avro` inerhits from and extends [Kafka::Producer](https://metacpan.org/pod/Kafka%3A%3AProducer).

# INSTALL

Installation of `Kafka::Producer::Avro` is a canonical:

    perl Makefile.PL
    make
    make test
    make install

## TEST NOTES

Tests are focused on verifying Avro-formatted messages and theirs interactions with Confluent Schema Registry and are intended to extend `Kafka::Producer` test suite.

They expect that in the target are listening Apache Kafka and Schema Registry services, respectively listening on `localhost:9092` and `http://localhost:8081`.

You can alternatively set a different URLs by exporting the following environment variable:

- `KAFKA_HOST`
- `KAFKA_PORT`
- `CONFLUENT_SCHEMA_REGISTY_URL`

For example:

    export KAFKA_HOST=my-kafka-host.my-domain.org
    export FALFA_PORT=9092
    export CONFLUENT_SCHEMA_REGISTY_URL=http://my-schema-registry-host.my-domain.org

# USAGE

## CONSTRUCTOR

### `new`

Creates new producer client object.

`new()` takes arguments in key-value pairs as described in [Kafka::Producer](https://metacpan.org/pod/Kafka%3A%3AProducer) from which it inherits.

In addition, takes in the following arguments:

- `SchemaRegistry => $schema_registry` (**mandatory**)

    Is a [Confluent::SchemaRegistry](https://metacpan.org/pod/Confluent%3A%3ASchemaRegistry) instance.

## METHODS

The following methods are defined for the `Kafka::Avro::Producer` class:

### `schema_registry`()

Returns the [Confluent::SchemaRegistry](https://metacpan.org/pod/Confluent%3A%3ASchemaRegistry) instance supplied to the construcor.

### `get_error`()

Returns a string containing last error message.

### `send( $topic, $partition, $messages, $keys, $compression_codec, $timestamps, $key_schema, $value_schema )`

### `send( %named_params )`

Sends Avro-formatted messages on a [Kafka::Connection](https://metacpan.org/pod/Kafka%3A%3AConnection) object.

Returns a non-blank value (a reference to a hash with server response description)
if the message is successfully sent.

In order to handle Avro format, `Kafka::Producer|Kafka::Producer` `send()` method is extended
with two more positional arguments, `$key_schema` and `$value_schema`:

    $producer->send(
          $topic,             # scalar 
          $partition,         # scalar
          $messages,          # scalar | array
          $keys,              # (optional) undef | scalar | array
          $compression_codec, # (optional) undef | scalar
          $timestamps,        # (optional) undef | scalar | array
          $key_schema,        # (optional) undef | JSON-string
          $value_schema       # (optional) undef | JSON-string
    );

Both `$key_schema` and `$value_schema` parameters are optional and must provide JSON strings that 
represent Avro schemas to use to validate and serialize key(s) and value(s).

These schemas are validated against `$schema_registry` and, if compliant, they are added to the registry
under the `$topic+'key'` or `$topic+'value'` Schema Registry's subjects.

If an expected schema isn't provided, latest version from Schema Registry is used accordingly to the  
(topic + key/value) subject. 

Alternatively, for ease of use, the `send()` method may be also used by suggesting named parameters:

    $producer->send(
          topic             => $topic,             # scalar 
          partition         => $partition,         # scalar
          messages          => $messages,          # scalar | array
          keys              => $keys,              # (optional) undef | scalar | array
          compression_codec => $compression_codec, # (optional) undef | scalar
          timestamps        => $timestamps,        # (optional) undef | scalar | array
          key_schema        => $key_schema,        # (optional) undef | JSON-string
          value_schema      => $value_schema       # (optional) undef | JSON-string
    );    

### `bulk_send( %params )`

Similar to `send` but uses bulks to avoid memory leaking.

Extra named parameters are expected:

- `size => $size`

    The size of the bulk

- `on_before_send_bulk => sub {...} ` (optional)

    A code block that will be executed before the sending of each bulk.

    The block will receive the following positional parameters: 

    - `$bulk_num` the number of the bulk
    - `$bulk_messages` the number of messages in the bulk
    - `$bulk_keys` the number of keys in the bulk
    - `$index_from` the absolute index of the first message in the bulk
    - `$index_to` the absolute index of the last message in the bulk

- `on_after_send_bulk => sub {...} ` (optional)

    A code block that will be executed after the sending of each bulk.

    The block will receive the following positional parameters: 

    - `$sent` the number of sent messages in the bulk
    - `$total_sent` the total number of messages sent

- `on_init => sub {...} ` (optional)

    A code block that will be executed only once before at the beginning of the cycle.

    The block will receive the following positional parameters: 

    - `$to_send` the total number of messages to send 
    - `$bulk_size` the size of the bulk

- `on_complete => sub {...} ` (optional)

    A code block that will be executed only once after the end of the cycle.

    The block will receive the following positional parameters: 

    - `$to_send` the total number of messages to send 
    - `$total_sent` the total number of messages sent
    - `$errors` the number bulks sent with errors

- `on_send_error => sub {...} ` (optional)

    A code block that will be executed when a bulk registers an error.

# AUTHOR

Alvaro Livraghi, <alvarol@cpan.org>

# CONTRIBUTE

[https://github.com/alivraghi/Kafka-Producer-Avro](https://github.com/alivraghi/Kafka-Producer-Avro)

# BUGS

Please use GitHub project link above to report problems or contact authors.

# COPYRIGHT AND LICENSE

Copyright 2018 by Alvaro Livraghi

This program is free software; you can redistribute it and/or modify it under the same terms as Perl itself.
