# Table of contents
1. [TCP header](#tcp-header)

## TCP header

| Field                    | Size (bits) |
|--------------------------|-------------|
| Source Port              | 16          |
| Destination Port         | 16          |
| Sequence Number          | 32          |
| Acknowledgment Number    | 32          |
| Data Offset (Header Len) | 4           |
| Reserved                 | 6           |
| Flags                    | 6           |
| Window Size              | 16          |
| Checksum                 | 16          |
| Urgent Pointer           | 16          |
| Options (if any)         | Variable    |
