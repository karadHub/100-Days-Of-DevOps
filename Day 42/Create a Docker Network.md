# Day 42: Create a Docker Network

## Task

The Nautilus DevOps team needs to set up several Docker environments for different applications. Create a Docker network based on the following requirements:

a. Create a Docker network named `beta` on App Server 1 in Stratos DC.
b. Configure it to use `macvlan` drivers.
c. Set it to use subnet `192.168.0.0/24` and IP range `192.168.0.0/24`.

---

## Solution

### Overview

This task involves creating a macvlan Docker network that allows containers to appear as physical devices on the network, providing direct access to the LAN infrastructure.

### Step-by-step Implementation

#### Step 1: Connect to App Server 1

```bash
ssh user@stapp01
# Replace 'user' with the actual username provided in your environment
```

#### Step 2: Check Available Network Interfaces

Before creating the macvlan network, identify the parent network interface:

```bash
ip a
# or
ip link show
```

Look for the primary network interface (commonly `eth0`, `ens33`, or similar).

#### Step 3: Create the Docker Network

```bash
docker network create -d macvlan \
  --subnet=192.168.0.0/24 \
  --ip-range=192.168.0.0/24 \
  --gateway=192.168.0.1 \
  -o parent=eth0 \
  beta
```

**Parameters Explained:**

- `-d macvlan`: Specifies the macvlan driver
- `--subnet=192.168.0.0/24`: Defines the subnet for the network
- `--ip-range=192.168.0.0/24`: Sets the IP range for container assignment
- `--gateway=192.168.0.1`: Sets the default gateway (adjust if needed)
- `-o parent=eth0`: Specifies the parent interface (replace `eth0` with your actual interface)
- `beta`: The name of the network

#### Step 4: Verify Network Creation

```bash
docker network ls
```

You should see the `beta` network listed with the `macvlan` driver.

#### Step 5: Inspect Network Details

```bash
docker network inspect beta
```

This command shows detailed configuration including subnet, gateway, and driver information.

---

## Alternative Commands (if gateway differs)

If the environment uses a different gateway, adjust accordingly:

```bash
# Example with different gateway
docker network create -d macvlan \
  --subnet=192.168.0.0/24 \
  --ip-range=192.168.0.0/24 \
  --gateway=192.168.0.254 \
  -o parent=eth0 \
  beta
```

---

## Verification and Testing

### Basic Verification

```bash
# List all networks
docker network ls | grep beta

# Detailed network inspection
docker network inspect beta | grep -E "(Subnet|IPRange|Gateway|Driver)"
```

### Optional: Test with a Container

```bash
# Create a test container using the beta network
docker run -it --network=beta --rm alpine sh

# Inside the container, check IP configuration
ip a
```

---

## Understanding Macvlan Networks

### What is Macvlan?

- **Macvlan** allows containers to have their own MAC addresses
- Containers appear as physical devices on the network
- Provides direct access to the LAN without NAT
- Useful for scenarios requiring container-to-physical-device communication

### Use Cases

- Legacy applications requiring direct network access
- Network monitoring tools
- Applications that need to broadcast on the LAN
- Services that require specific IP addresses

### Limitations

- Cannot communicate with the Docker host directly
- Requires promiscuous mode support on the network interface
- May not work in all virtualized environments

---

## Troubleshooting

### Common Issues and Solutions

1. **Permission Denied**

   ```bash
   sudo docker network create -d macvlan \
     --subnet=192.168.0.0/24 \
     --ip-range=192.168.0.0/24 \
     --gateway=192.168.0.1 \
     -o parent=eth0 \
     beta
   ```

2. **Interface Not Found**

   - Check available interfaces: `ip link show`
   - Use the correct interface name in the `parent` option

3. **Network Already Exists**

   ```bash
   # Remove existing network
   docker network rm beta
   # Then recreate with the correct configuration
   ```

4. **Subnet Conflicts**
   - Ensure the specified subnet doesn't conflict with existing networks
   - Check with: `docker network ls` and `ip route`

---

## Best Practices

1. **Plan IP Ranges**: Ensure the IP range doesn't conflict with existing infrastructure
2. **Document Networks**: Keep track of created networks and their purposes
3. **Test Connectivity**: Always verify network functionality before production use
4. **Security Considerations**: Macvlan networks bypass Docker's built-in isolation

---

## Real-World Applications

This type of network setup is commonly used for:

- **Microservices Architecture**: Providing direct network access to services
- **Legacy System Integration**: Connecting containerized apps to existing infrastructure
- **Network Function Virtualization (NFV)**: Running network services in containers
- **IoT Applications**: Containers that need to communicate with physical devices

The `beta` network is now ready for use by applications that require direct network access in the Stratos Datacenter environment.
