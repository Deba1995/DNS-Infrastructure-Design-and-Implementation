# DNS-Infrastructure-Design-and-Implementation

# Enterprise DNS Infrastructure with BIND9

## What I Built

This project demonstrates a DNS infrastructure using BIND9 on Ubuntu. I set up primary and secondary DNS servers with security features including split-horizon DNS, TSIG-authenticated zone transfers, and dynamic DNS updates.

The setup simulates an environment where internal users need access to sensitive systems while external users get limited DNS information for security.

## Network Setup

- **Primary DNS Server**: dns1 (172.28.1.213)
- **Secondary DNS Server**: dns2 (172.24.1.214) 
- **Domain**: gentech.solution
- **Internal Network**: 172.28.1.0/24, 172.24.1.0/24
- **External Network**: 172.27.1.0/24

## Key Features I Implemented

### Split-Horizon DNS (Views)
Different DNS responses based on where the query comes from. Internal users can resolve sensitive hosts like `vault.gentech.solution`, but external users get NXDOMAIN for the same query. This is critical for security in real environments.

### TSIG-Secured Zone Transfers
Zone transfers between primary and secondary servers are authenticated using TSIG keys. Without the correct key, transfer requests are denied and logged as security events.

### Dynamic DNS Updates
Authenticated dynamic updates allow authorized systems to add/modify DNS records automatically. I tested both successful updates with proper keys and blocked attempts without authentication.

### Comprehensive Logging
All DNS operations are logged including queries, zone transfers, security events, and dynamic updates. This provides complete audit trails required in production environments.

## Architecture

The infrastructure uses view-based security where:
- Internal clients get full zone access
- External clients get limited public records only
- Zone transfers happen securely between DNS servers
- Dynamic updates require proper authentication

## Testing Results

I thoroughly tested every security feature:

### Internal vs External Access
- Internal clients can resolve `vault.gentech.solution` → `172.24.1.130`
- External clients get `NXDOMAIN` for the same query
- Both can access basic domain information like SOA records

### Zone Transfer Security
- Secondary server successfully transfers zones using TSIG authentication
- Unauthorized transfer attempts from clients are blocked and logged
- Transfer logs show proper AXFR and IXFR operations

### Dynamic DNS Security
- Updates with proper TSIG key succeed and create journal files
- Updates without authentication fail with `REFUSED` status
- All attempts are logged with client IP addresses

Full test outputs are available in the `logs/test-outputs/` directory.

## File Structure
```
├── configs
│   ├── keys
│   │   └── zone-xfer.key.example
│   ├── primary
│   │   ├── zones
│   │   │   ├── forward
│   │   │   │   ├── db.gentech.solution.external
│   │   │   │   └── db.gentech.solution.internal
│   │   │   └── reverse
│   │   │       ├── db.172.24.1.internal
│   │   │       └── db.172.28.1.internal
│   │   ├── named.conf.local
│   │   └── named.conf.options
│   └── secondary
│       ├── named.conf.local
│       └── named.conf.options
├── logs
│   └── test-outputs
│       ├── ddns-tests.txt
│       ├── external-client-tests.txt
│       ├── general.log.sample
│       ├── internal-client-tests.txt
│       └── zone-transfer-tests.txt
└── README.md
```


## Security Implementation

### Access Control
- ACL-based view matching prevents unauthorized access
- TSIG authentication protects zone transfers and updates
- Rate limiting prevents abuse from repeated failed requests
- Security events logged for forensic analysis

## What I Learned

Working on this project taught me how enterprise DNS really works beyond basic resolution. The security layers like split-horizon views and TSIG authentication are crucial for protecting internal infrastructure while still providing necessary external services.

The logging and monitoring aspects showed me how important audit trails are for production systems. Every security event is tracked with timestamps and client information.

Most challenging part was getting the file permissions right for dynamic DNS. BIND needs specific ownership to create and manage journal files for zone updates.

## Real-World Applications

This setup would work in any organization that needs:
- Secure internal DNS with external public presence
- Automated DNS updates from applications or DHCP
- High availability with primary/secondary architecture
- Complete audit logging for compliance requirements

---
