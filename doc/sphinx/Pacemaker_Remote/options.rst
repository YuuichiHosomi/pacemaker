Configuration Explained
-----------------------

The walk-through examples use some of these options, but don't explain exactly
what they mean or do.  This section is meant to be the go-to resource for all
the options available for configuring pacemaker_remote-based nodes.

.. index::
    single: configuration

Resource Meta-Attributes for Guest Nodes
########################################

When configuring a virtual machine as a guest node, the virtual machine is
created using one of the usual resource agents for that purpose (for example,
ocf:heartbeat:VirtualDomain or ocf:heartbeat:Xen), with additional metadata
parameters.

No restrictions are enforced on what agents may be used to create a guest node,
but obviously the agent must create a distinct environment capable of running
the pacemaker_remote daemon and cluster resources. An additional requirement is
that fencing the host running the guest node resource must be sufficient for
ensuring the guest node is stopped. This means, for example, that not all
hypervisors supported by VirtualDomain may be used to create guest nodes; if
the guest can survive the hypervisor being fenced, it may not be used as a
guest node.

Below are the metadata options available to enable a resource as a guest node
and define its connection parameters.

.. table:: **Meta-attributes for configuring VM resources as guest nodes**

  +------------------------+-----------------+-----------------------------------------------------------+
  | Option                 | Default         | Description                                               |
  +========================+=================+===========================================================+
  | remote-node            | none            | The node name of the guest node this resource defines.    |
  |                        |                 | This both enables the resource as a guest node and        |
  |                        |                 | defines the unique name used to identify the guest node.  |
  |                        |                 | If no other parameters are set, this value will also be   |
  |                        |                 | assumed as the hostname to use when connecting to         |
  |                        |                 | pacemaker_remote on the VM.  This value **must not**      |
  |                        |                 | overlap with any resource or node IDs.                    |
  +------------------------+-----------------+-----------------------------------------------------------+
  | remote-port            | 3121            | The port on the virtual machine that the cluster will     |
  |                        |                 | use to connect to pacemaker_remote.                       |
  +------------------------+-----------------+-----------------------------------------------------------+
  | remote-addr            | 'value of'      | The IP address or hostname to use when connecting to      |
  |                        | ``remote-node`` | pacemaker_remote on the VM.                               |
  +------------------------+-----------------+-----------------------------------------------------------+
  | remote-connect-timeout | 60s             | How long before a pending guest connection will time out. |
  +------------------------+-----------------+-----------------------------------------------------------+

Connection Resources for Remote Nodes
#####################################

A remote node is defined by a connection resource. That connection resource
has instance attributes that define where the remote node is located on the
network and how to communicate with it.

Descriptions of these instance attributes can be retrieved using the following
``pcs`` command:

.. code-block:: none

    # pcs resource describe remote
    ocf:pacemaker:remote - remote resource agent

    Resource options:
      server: Server location to connect to. This can be an ip address or hostname.
      port: tcp port to connect to.
      reconnect_interval: Interval in seconds at which Pacemaker will attempt to
                          reconnect to a remote node after an active connection to
                          the remote node has been severed. When this value is
                          nonzero, Pacemaker will retry the connection
                          indefinitely, at the specified interval.

When defining a remote node's connection resource, it is common and recommended
to name the connection resource the same as the remote node's hostname. By
default, if no **server** option is provided, the cluster will attempt to contact
the remote node using the resource name as the hostname.

Example defining a remote node with the hostname **remote1**:

.. code-block:: none

    # pcs resource create remote1 remote

Example defining a remote node to connect to a specific IP address and port:

.. code-block:: none

    # pcs resource create remote1 remote server=192.168.122.200 port=8938

Environment Variables for Daemon Start-up
#########################################

Authentication and encryption of the connection between cluster nodes
and nodes running pacemaker_remote is achieved using
with `TLS-PSK <https://en.wikipedia.org/wiki/TLS-PSK>`_ encryption/authentication
over TCP (port 3121 by default). This means that both the cluster node and
remote node must share the same private key. By default, this
key is placed at ``/etc/pacemaker/authkey`` on each node.

You can change the default port and/or key location for Pacemaker and
pacemaker_remote via environment variables. How these variables are set varies
by OS, but usually they are set in the ``/etc/sysconfig/pacemaker`` or
``/etc/default/pacemaker`` file.

.. code-block:: none

    #==#==# Pacemaker Remote
    # Use a custom directory for finding the authkey.
    PCMK_authkey_location=/etc/pacemaker/authkey
    #
    # Specify a custom port for Pacemaker Remote connections
    PCMK_remote_port=3121

Removing Remote Nodes and Guest Nodes
#####################################

If the resource creating a guest node, or the **ocf:pacemaker:remote** resource
creating a connection to a remote node, is removed from the configuration, the
affected node will continue to show up in output as an offline node.

If you want to get rid of that output, run (replacing $NODE_NAME appropriately):

.. code-block:: none

    # crm_node --force --remove $NODE_NAME

.. WARNING::

    Be absolutely sure that there are no references to the node's resource in the
    configuration before running the above command.
