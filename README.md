
# National Cybersecurity CTF Hackathon: Operation Trace Walkthrough  
*Forensic Analysis Challenge Solution*

## Challenge Overview
Operation Trace simulates a compromised system (Node #47) where cold wallets were drained and logs vanished. The objective is to recover a fragmented trace left by an analyst under duress, involving:
- Steganography analysis
- File system forensics
- Cryptographic reconstruction
- Disk image recovery

**Final Flag**: `flag{this_is_the_last_trace, you did it. please submit me}`

---

## Tools Used
```
- `unzip` - Archive extraction
- `exiftool` - Image metadata analysis
- `binwalk` - Embedded data detection
- `steghide` - Steganography extraction
- `strings` - Binary string extraction
- `grep` - Pattern matching
- `base64` - Data decoding
- `fdisk` - Disk image analysis
- `chmod` - File permission management
```

---

## Stage 1: Initial Extraction & Image Analysis
### Key Steps:
1. Unzipped the initial archive containing 10 trace images (`trace_0035.jpg` to `trace_0043.jpg`)
2. Checked the README.txt present in the stage1 folder and followed this link  (https://www.youtube.com/watch?v=E3Q46qdvBD4) to watch the video attached 
3. Identified duplicate images in the folder and verified using metadata analysis:
   ```command
   exiftool trace_*.jpg
   binwalk trace_0042.jpg
   ```
4. Extracted hidden password from `trace_0042.jpg`:
   ```command
   steghide extract -sf trace_0042.jpg  # No passphrase needed
   ```
   **Output**: `password.txt` containing decryption key for next phase

---

## Stage 2: Passphrase Reconstruction
**Unlocked  both Stage 2 and Stage 3 Folder with the passphrase gotten in the Stage 1**
### Artifacts Recovered:
- `decode_phrase` (byte-compiled Python module)
- `index.html` (terminal interface)
- System logs

### Critical Findings:
1. `index.html` revealed partial passphrase:  
   `TRACE-PHRASE-FRAGMENT: alpha|omega`
2. System logs indicated corruption at **03:22**:  
   `2025-02-17 03:22:07 [WARN] TRACE fragment incomplete`
3. Recovered full passphrase using:
   ```bash
   strings decode_phrase | grep -A 5 'phrase'
   ```
   **Output**: `alpha`, `omega`, `mission`

4. Executed reconstruction script:
   ```bash
   chmod +x decode_phrase
   ./decode_phrase
   ```
   ```python
   Enter phrase part 1: alpha
   Enter phrase part 2: omega
   Enter phrase part 3: mission
   ```
   **Output**: `Override PIN > 897441`

---

## Phase 3: Disk Image Analysis & Flag Extraction
**Navigate through this stage's folder, meet with alot of decoys till i discoverd two corrupted disk images in a hidden file**
### Forensic Process:
1. Located corrupted disk image:  
   `~/.stage3_recovered/.diskimage.dd`
2. Scanned both for encoded payloads
3. Only one give the desired findings :
   ```bash
   strings -n 10 .diskimage.dd | grep -E '[A-Za-z0-9+/]{20,}={0,2}' | head
   ```
   **Output**:  
   `ZmxhZ3t0a6LZX21ZX3RoZV9SYXN0X3RyYWN1LCB5b3UgZGLKIGl0LiBwb6Vhc2Ugc3VibWl0I61lfQ==`

4. Decoded base64 payload:
   ```bash
   echo "ZmxhZ3t0a6LZX21ZX3RoZV9...I61lfQ==" | base64 -d
   ```
   **Flag**: `flag{this_is_the_last_trace, you did it. please submit me}`

---

## Key Forensic Insights
1. **Steganography**: Attackers hid keys in image metadata
2. **Fragmented Artifacts**: Corrupted logs required cross-referencing timestamps (03:22)
3. **Data Reconstruction**:
   - Passphrase fragments in HTML comments
   - Disk remnants in unallocated space
4. **Anti-Forensic Tactics**:
   - Decoy files (`6-digit`, `agent.log`)
   - Byte-compiled scripts to obscure logic

---

## Conclusion
This challenge demonstrated real-world forensic techniques for:
1. Recovering fragmented evidence
2. Bypassing anti-forensic measures
3. Reconstructing cryptographic materials
The solution required correlating timestamps, file metadata, and binary analysis to recover the final flag.
