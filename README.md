# Overview

This interface layer handles the communication between a Livy client and Livy.

# Usage

## Provides

The Livy deployment is provided by the Livy charm. this charm has to
signal to all its related clients that has become available.

The interface layer sets the following state as soon as a client is connected:

  * `{relation_name}.joined` The relation between the client and Livy is established.

The Livy provider can signal its availability through the following methods:

  * `set_ready()` Livy is available.

  * `clear_ready()` Livy is not available.

  * `send_port()` Sends port over relation

  * `send_hostname()` Sends hostname over relation

An example of a charm using this interface would be:

```python
@when('livy.started', 'client.related')
def client_present(client):
    client.set_ready()


@when('client.related')
@when_not('livy.started')
def client_should_stop(client):
    client.clear_ready()
```


## Requires

This is the side that a Livy client charm (e.g., HUE)
will use to be informed of the availability of Livy.

The interface layer will set the following state for the client to react to, as
appropriate:

  * `{relation_name}.joined` The client is related to Livy and is waiting for Livy to become available.

  * `{relation_name}.ready` Livy is ready to be used.

An example of a charm using this interface would be:

```python
@when('hue.installed', 'livy.ready')
@when_not('hue.started')
def configure_hue(livy):
    hookenv.status_set('maintenance', 'Setting up Hue')
    hue = Hue(get_dist_config())
    hue.start()
    set_state('hue.started')
    hookenv.status_set('active', 'Ready')


@when('hue.started')
@when_not('livy.ready')
def stop_hue():
    hue = Hue(get_dist_config())
    hue.stop()
    remove_state('hue.started')
```


# Contact Information

- <bigdata@lists.ubuntu.com>

