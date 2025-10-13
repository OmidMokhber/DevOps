# Newer versions of the CRC executable require manual set up to prevent potential incompatibilities with earlier versions.
Procedure
1. [Download the latest CRC release.](https://console.redhat.com/openshift/create/local)
2. Save any desired information stored in your existing instance.
3. Delete the existing CRC instance.
```bash
crc delete
```
4. Replace the earlier crc executable with the executable of the latest release. Verify that the new crc executable is in use by checking its version:
```bash
crc version
```
5. Set up the new CRC release:
```bash
crc setup
```
6. Start the new CRC instance:
```bash
crc start
```
