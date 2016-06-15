

###[**Implementing dm-verity**](http://source.android.com/security/verifiedboot/dm-verity.html)

> **Operation**

dm-verity protection lives in the kernel. So if rooting software compromises the system before the kernel comes up, it will retain that access. To mitigate this risk, most manufacturers verify the kernel using a key burned into the device. That key is not changeable once the device leaves the factory.

Manufacturers use that key to verify the signature on the first-level bootloader, which in turn verifies the signature on subsequent levels, the application bootloader and eventually the kernel. Each manufacturer wishing to take advantage of verified boot should have a method for verifying the integrity of the kernel. Assuming the kernel has been verified, the kernel can look at a block device and verify it as it is mounted.

One way of verifying a block device is to directly hash its contents and compare them to a stored value. However, attempting to verify an entire block device can take an extended period and consume much of a device's power. Devices would take long periods to boot and then be significantly drained prior to use.

Instead, dm-verity verifies blocks individually and only when each one is accessed. When read into memory, the block is hashed in parallel. The hash is then verified up the tree. And since reading the block is such an expensive operation, the latency introduced by this block-level verification is comparatively nominal.

If verification fails, the device generates an I/O error indicating the block cannot be read. It will appear as if the filesystem has been corrupted, as is expected.

Applications may choose to proceed without the resulting data, such as when those results are not required to the application's primary function. However, if the application cannot continue without the data, it will fail.