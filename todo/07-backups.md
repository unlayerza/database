# Phase 07 — Snapshots, Backups and Bunny Storage

## Snapshots

- [ ] Consistent snapshot creation
- [ ] Snapshot sequence metadata
- [ ] Term metadata
- [ ] Database ID
- [ ] Engine/schema versions
- [ ] Checksum
- [ ] Snapshot verification
- [ ] Local restore

## Backup pipeline

- [ ] Compression
- [ ] Encryption
- [ ] Checksum
- [ ] Bunny Storage upload
- [ ] Remote metadata
- [ ] Retry
- [ ] Remote verification
- [ ] Download
- [ ] Decrypt
- [ ] Decompress
- [ ] Restore

## Retention

- [ ] Hourly snapshots
- [ ] 48-hour hourly retention
- [ ] Hourly expiration
- [ ] Daily recovery points
- [ ] Daily retention
- [ ] Extended paid retention model
- [ ] Protected recovery points

## Verification

- [ ] Checksum after upload
- [ ] Download verification
- [ ] Decryption verification
- [ ] Database integrity verification
- [ ] Automated restore test

## Acceptance

- [ ] Live database produces valid remote backup without unsafe file copying
- [ ] Remote backup restores into a fresh database
