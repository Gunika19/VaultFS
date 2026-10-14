# VaultFS: Encrypted Block-Based File System

VaultFS is a file system simulation written in C++. It stores everything on a single binary file that acts as a virtual disk, and it manages files the way a real file system does: with inodes, a SuperBlock, a free-block bitmap and fixed-size blocks. File contents are encrypted with AES-256-GCM, so data on the "disk" is unreadable without the password and any tampering is detected on read.

It started as an Operating Systems course project (ULC404, Thapar Institute of Engineering and Technology) built with Devansh Gupta, where files were protected with a simple XOR cipher. This version replaces that with real authenticated encryption and extends the design.

## Features

- **Block-based storage:** the virtual disk is divided into fixed 512-byte blocks. Files can span multiple blocks.
- **Metadata management:** an inode table stores each file's name, size, block pointers and usage status. A SuperBlock tracks total blocks and free space.
- **Free-block bitmap:** allocates and releases blocks as files are created, written and deleted.
- **Authenticated encryption:** file contents are encrypted with AES-256-GCM (OpenSSL). The key is derived from the user's password with PBKDF2 using a random salt. Each write uses a fresh random nonce, and the authentication tag is stored with the file's metadata. A modified block fails verification when read.
- **Tag-based search:** assign a tag to a file and list every file with a given tag.
- **Persistence:** all metadata and data are saved to the disk file, so the file system survives across sessions.
- **CLI:** menu-driven interface for formatting the disk, creating, writing, reading and deleting files, and searching by tag.

## Design

### On-disk layout

```
+------------+---------------+--------------+-------------------------+
| SuperBlock |  Free-block   |  Inode table |      Data blocks        |
|            |    bitmap     |              |   (512 bytes each)      |
+------------+---------------+--------------+-------------------------+
```

- **SuperBlock:** total block count, free block count, and the PBKDF2 salt.
- **Inode:** file name, size, tag, block pointers, in-use flag, GCM nonce and authentication tag.
- **Data blocks:** hold the encrypted file contents.

### Encryption flow

1. On startup the user enters a password. PBKDF2 turns it, together with the salt from the SuperBlock, into a 256-bit key.
2. **Write:** a random nonce is generated, the data is encrypted with AES-256-GCM, the ciphertext goes to the allocated blocks, and the nonce and tag are saved in the inode.
3. **Read:** the ciphertext is read from the blocks and decrypted with the stored nonce. If the tag does not verify (wrong password or modified data), the read fails instead of returning corrupted data.

## Build and run

Requirements: a C++17 compiler and OpenSSL development headers.

```bash
# Ubuntu / Debian
sudo apt install build-essential libssl-dev

# Build
g++ -std=c++17 -o vaultfs src/*.cpp -lcrypto

# Run
./vaultfs
```

```
1. Format Disk
2. Create File
3. Write File
4. Read File
5. Delete File
6. Search By Tag
7. Exit
```

> Adjust the build command above to match your repository layout.

## Project structure

```
vaultfs/
├── src/          # file system, encryption and CLI source files
├── README.md
└── LICENSE
```

> Update this tree to match your actual files.

## Limitations

- VaultFS is a user-space simulation on a single disk file. It is not a kernel file system and is not meant to store real sensitive data.
- The cryptography relies on OpenSSL and has not been independently audited.
- Directories, permissions and journaling are not implemented yet.

## Roadmap

- Hierarchical directories with owner and permission bits
- Write-ahead journaling for crash recovery
- Unit tests and a crash-recovery test suite

## Acknowledgments

Original file system design and XOR-based prototype: Gunika Sohal and Devansh Gupta, Operating Systems project (ULC404), Thapar Institute of Engineering and Technology, 2024-25.

## License

Add your preferred license (for example MIT) as a `LICENSE` file.
