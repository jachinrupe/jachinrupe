# Interface Remapping Notes (Cisco 7200 in GNS3)

Cisco 7200 interface numbering depends on the modules in Slots.

If a config references FastEthernet1/0 but your router only has FastEthernet0/0, do:

1. Check available interfaces:
   show ip int brief

2. Update the config by replacing the interface names to match what you have.

Common patterns:
- Some images/modules produce only Fa0/0 unless you add a second FE module.
- Adding PA-FE-TX to another slot typically creates Fa1/0.


## Your current mapping (from show ip int brief)
- PE1: Fa0/0=10.0.10.2 (CE), Fa1/0=10.0.12.1 (core)
- PE2: Fa0/0=10.0.23.1 (core), Fa1/0=10.0.20.2 (CE)
