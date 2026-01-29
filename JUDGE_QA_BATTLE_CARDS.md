# 🎤 FIRM-LOCK: Judge Q&A Battle Cards

> **50+ Anticipated Questions with Killer Answers**

---

## 🎯 How to Use These Cards

1. **Read through all questions** before the presentation
2. **Practice your answers** until they feel natural
3. **Assign team members** to specific question categories
4. **Have backup data ready** (numbers, screenshots, demos)

---

# 📚 QUESTION CATEGORIES

## Category 1: Technical Deep-Dive

### Q1: "Walk me through the attestation protocol step-by-step."

**🎯 Your Answer:**
> "Absolutely. The protocol is a challenge-response with four security guarantees:
> 
> **Step 1:** Verifier generates a random 32-byte nonce + timestamp and sends it to the device.
> 
> **Step 2:** Device collects its PCR values - these are cryptographic fingerprints of every piece of code that ran during boot.
> 
> **Step 3:** Device concatenates nonce + PCRs + firmware version, hashes it, and signs with its private key stored in the secure element.
> 
> **Step 4:** Verifier receives the evidence, verifies the signature against the device's public key, checks the nonce is fresh, and compares PCRs to golden values.
> 
> **Step 5:** Decision - PASS if everything matches, FAIL if anything's off.
> 
> The whole exchange takes under 2 seconds over LoRa."

**💡 Pro Tip:** Draw the flow on a whiteboard if available.

---

### Q2: "What happens if the secure element is physically attacked?"

**🎯 Your Answer:**
> "The ATECC608A has multiple hardware countermeasures:
> 
> • **Voltage glitch detection** - shuts down if power is manipulated
> • **Temperature monitoring** - triggers at abnormal temps
> • **Side-channel resistance** - power analysis won't leak keys
> • **Tamper sensors** - detects physical probing attempts
> 
> But here's the key: even if someone extracted the key, they'd only compromise ONE device. Each device has a unique key generated during manufacturing. Compare that to software solutions where one leaked key compromises your entire fleet.
> 
> For defense applications, we can upgrade to the SE050 - it's Common Criteria EAL 6+ certified, the same level used in military hardware."

---

### Q3: "How do you prevent replay attacks?"

**🎯 Your Answer:**
> "Two mechanisms:
> 
> **1. Nonce Freshness:** Every challenge includes a unique, random 32-byte nonce. The device MUST include this exact nonce in its signed response. The verifier tracks used nonces and rejects any replay.
> 
> **2. Timestamp Validation:** Challenges expire after 60 seconds. An old response won't be accepted even if the nonce is fresh.
> 
> We tested this - I can show you the code that rejects replay attempts."

---

### Q4: "What's the performance overhead?"

**🎯 Your Answer:**
> "Minimal. Here are our measured numbers:
> 
> | Metric | Unsigned Boot | Signed Boot | Overhead |
> |--------|---------------|-------------|----------|
> | Boot Time | 120ms | 580ms | +460ms |
> | Attestation | N/A | 1.2s | - |
> | Memory | 45KB | 52KB | +7KB |
> | Power (sleep) | 9µA | 9µA | 0 |
> 
> The 460ms boot overhead is for signature verification - happens once per boot. Attestation is on-demand. And sleep power is identical because we use the secure element's ultra-low-power mode.
> 
> For context: 460ms is less time than it takes to blink."

---

### Q5: "Why SHA-256 and ECDSA P-256? Why not something faster?"

**🎯 Your Answer:**
> "Three reasons:
> 
> **1. Hardware Acceleration:** The STM32U5 has dedicated SHA-256 and ECC hardware accelerators. SHA-256 is actually faster than SHA-1 on this chip.
> 
> **2. Standards Compliance:** NIST SP 800-193 recommends SHA-256 minimum. Using weaker algorithms would fail certification.
> 
> **3. Future-Proofing:** SHA-1 is broken. We don't want to redesign in 2 years.
> 
> The ATECC608A does ECDSA signing in ~50ms - that's hardware-accelerated and unextractable. Can't beat that."

---

### Q6: "How do you handle firmware updates?"

**🎯 Your Answer:**
> "Secure OTA with three protections:
> 
> **1. Signature Verification:** Updates must be signed with our firmware signing key. Unsigned updates are rejected.
> 
> **2. Anti-Rollback:** The secure element has a monotonic counter. We reject any firmware with a version number lower than the counter. Prevents downgrade attacks.
> 
> **3. Atomic Swap:** Updates are written to a staging area first. Only after successful verification do we swap primary and staging. If power fails mid-update, the device boots the old firmware.
> 
> **4. Recovery Mode:** If new firmware fails to boot 3 times, we automatically roll back to the golden image."

---

### Q7: "What about side-channel attacks on the MCU itself?"

**🎯 Your Answer:**
> "Good question. The STM32U5 has several protections:
> 
> • **TrustZone** isolates secure code from normal code
> • **Hardware crypto** prevents timing attacks on software implementations
> • **JTAG disable** prevents debug access in production
> • **Flash protection** prevents readout
> 
> But you're right - the MCU is more vulnerable than the secure element. That's why:
> • Private keys NEVER touch the MCU - they stay in the ATECC608A
> • The MCU only handles public data (PCRs, nonces)
> • If MCU is compromised, attacker learns nothing about keys
> 
> It's defense in depth."

---

### Q8: "How do you verify the bootloader itself?"

**🎯 Your Answer:**
> "Two-stage verification:
> 
> **Stage 0:** The STM32U5 has an immutable ROM bootloader. It can be configured to verify the Stage 1 bootloader signature before executing. This is done via OTP fuses - once set, can't be changed.
> 
> **Stage 1:** Our MCUboot bootloader is signed. On every boot, it:
> 1. Measures itself (extends PCR[0])
> 2. Verifies its own signature via the secure element
> 3. Only then proceeds to verify the application
> 
> The bootloader is also stored in a protected flash region that can't be modified without special privileges.
> 
> We also have a 'measured boot' chain - each stage measures the next before executing."

---

### Q9: "What's the memory footprint?"

**🎯 Your Answer:**
> "Breakdown for STM32U5 (2MB flash):
> 
> | Region | Size | Purpose |
> |--------|------|---------|
> | Bootloader | 128KB | MCUboot + SE driver |
> | Primary App | 128KB | Main firmware |
> | Staging | 128KB | Update area |
> | Golden Image | 128KB | Factory recovery |
> | Audit Logs | 4KB | Security events |
> | **Total Used** | **516KB** | **26% of flash** |
> 
> We have plenty of room for application code. The 2MB STM32U5 was intentional - we wanted headroom for future features without changing hardware."

---

### Q10: "How do you handle clock drift for timestamps?"

**🎯 Your Answer:**
> "The 60-second nonce expiration is loose enough to handle reasonable drift. But for stricter requirements:
> 
> • Devices can sync time during attestation (verifier provides timestamp)
> • We support external RTC crystals for better accuracy
> • For air-gapped devices, we use relative time (counters) instead of absolute timestamps
> 
> In practice, the STM32U5's internal RTC is accurate to ±20ppm - that's about 2 seconds per day. Well within our tolerance."

---

## Category 2: Business & Market

### Q11: "Who's your target customer?"

**🎯 Your Answer:**
> "Three primary markets:
> 
> **1. Defense & Government ($2B market)**
>    • Border monitoring sensors
>    • Tactical communication devices
>    • Forward operating base infrastructure
>    • DoD Zero Trust mandate creates urgency
> 
> **2. Critical Infrastructure ($5B market)**
>    • Power grid SCADA systems
>    • Water treatment facilities
>    • Oil & gas pipeline sensors
>    • NIS2 compliance requirement
> 
> **3. Medical Devices ($3B market)**
>    • Implantable devices
>    • Hospital equipment
>    • FDA cybersecurity guidance
> 
> We've got 2 LOIs from defense contractors and are in talks with a medical device OEM."

---

### Q12: "What's your revenue model?"

**🎯 Your Answer:**
> "Three streams:
> 
> **1. Hardware Sales (40% margin)**
>    • $18/unit at 1k volume
>    • $14/unit at 10k volume
>    • Target: $8M ARR at 100k units/year
> 
> **2. Verifier SaaS ($1/device/month)**
>    • Cloud dashboard
>    • Fleet management
>    • Compliance reporting
>    • Target: $12M ARR at 100k devices
> 
> **3. Enterprise Support**
>    • Custom integrations
>    • Certification assistance
>    • Training & consulting
>    • Target: $2M ARR
> 
> **Total Target: $22M ARR at scale**
> 
> We're also pursuing SBIR grants - $250k non-dilutive funding for defense R&D."

---

### Q13: "How big is the market?"

**🎯 Your Answer:**
> "TAM/SAM/SOM analysis:
> 
> **TAM: $16B** (IoT Security + Defense IoT)
> • Growing 25% YoY
> • Driven by regulation and threat landscape
> 
> **SAM: $800M** (Firmware Security Niche)
> • Industrial, medical, defense
> • Devices requiring offline operation
> 
> **SOM: $50M** (Year 5 Target)
> • 2% of SAM
> • 500k devices at $100 average revenue
> 
> The real opportunity is timing. EU NIS2, DoD Zero Trust, FDA rules - all mandate device integrity. Companies NEED a solution like ours."

---

### Q14: "Who are your competitors?"

**🎯 Your Answer:**
> "Three categories:
> 
> **1. Cloud TPM (Microsoft Azure, AWS)**
>    • ❌ Require constant internet
>    • ❌ Expensive ($50-200/device)
>    • ❌ Complex integration
>    • ✅ Enterprise-grade security
> 
> **2. Hardware TPM (Infineon, NXP)**
>    • ❌ Overkill for IoT (server-focused)
>    • ❌ High power consumption
>    • ❌ Complex drivers
>    ✅ Tamper-resistant
> 
> **3. Proprietary (Cisco, Intel)**
>    • ❌ Vendor lock-in
>    • ❌ Closed source
>    • ❌ Can't customize
>    • ✅ Integrated solutions
> 
> **FIRM-LOCK is the ONLY solution that's:**
> ✅ Offline-capable
> ✅ MCU-optimized
> ✅ <$20/device
> ✅ Open source"

---

### Q15: "What's stopping a big company from copying you?"

**🎯 Your Answer:**
> "Three moats:
> 
> **1. Time to Market:**
>    We've spent 6 months integrating MCUboot + ATECC608A + LoRa. 
>    A competitor starting today would need 12-18 months to catch up.
> 
> **2. Certifications:**
>    We're pursuing PSA Certified Level 2 and SESIP Level 3.
>    These take 6-12 months and significant investment.
>    Once certified, we have a stamp of approval competitors lack.
> 
> **3. Network Effects:**
>    We're building an open-source community.
>    More users = more contributors = better security.
>    Network effects in security are powerful.
> 
> **4. Patentable IP:**
>    Our LoRa-based attestation protocol is novel.
>    We're filing provisional patents."

---

### Q16: "How do you plan to scale manufacturing?"

**🎯 Your Answer:**
> "Three-phase approach:
> 
> **Phase 1 (Now):** Prototype
>    • Off-the-shelf dev boards
>    • Hand-assembled
>    • For pilots and demos
> 
> **Phase 2 (6 months):** Low Volume
>    • Custom PCB design
>    • JLCPCB or similar (100-1000 units)
>    • Manual assembly + reflow
> 
> **Phase 3 (12 months):** High Volume
>    • Partner with EMS (Electronics Manufacturing Service)
>    • Foxconn, Jabil, or regional equivalent
>    • Automated assembly, testing, provisioning
> 
> The key is the secure element provisioning. We need a secure facility for initial key injection. We've identified partners who do this for smart card manufacturing."

---

### Q17: "What's your customer acquisition strategy?"

**🎯 Your Answer:**
> "Three channels:
> 
> **1. Direct Sales (Defense/Gov)**
>    • SBIR program participation
>    • Trade shows (AUSA, CyberSat)
>    • Direct outreach to CISOs
> 
> **2. Channel Partners (Enterprise)**
>    • System integrators (Lockheed, Northrup)
>    • IoT platform vendors
>    • MSPs (Managed Service Providers)
> 
> **3. Developer Community (Open Source)**
>    • GitHub presence
>    • Technical blog posts
>    • Conference talks
>    • Reference designs
> 
> Our open-source approach is intentional - developers adopt, recommend to management, and drive enterprise sales."

---

### Q18: "What are your unit economics?"

**🎯 Your Answer:**
> "At 1,000 unit scale:
> 
> **COGS: $17.65**
> • MCU: $8.50
> • Secure Element: $0.58
> • LoRa Module: $4.20
> • PCB + Passives: $4.37
> 
> **Price: $35** (2x COGS)
> **Gross Margin: 50%**
> 
> At 10,000 units:
> • COGS drops to $14.20
> • Price drops to $28
> • Margin stays at 50%
> 
> The SaaS is pure margin - $1/device/month with minimal hosting costs.
> 
> **Break-even: 500 devices/month**"

---

## Category 3: Security & Trust

### Q19: "How do we know YOUR code isn't backdoored?"

**🎯 Your Answer:**
> "Three answers:
> 
> **1. Open Source:**
>    Our entire stack is open source. Anyone can audit.
>    We're on GitHub with full commit history.
> 
> **2. Reproducible Builds:**
>    Same source code always produces same binary.
>    You can verify our releases match the source.
> 
> **3. Third-Party Audit:**
>    We're engaging a security firm for penetration testing.
>    Results will be published.
> 
> **4. Supply Chain:**
>    We use standard components from authorized distributors.
>    No custom silicon, no hidden functionality.
> 
> Compare to proprietary solutions - you have to trust the vendor. With us, you can verify."

---

### Q20: "What if your signing key is stolen?"

**🎯 Your Answer:**
> "Key management is critical. Our approach:
> 
> **1. HSM Storage:**
>    Firmware signing keys are stored in Hardware Security Modules.
>    Keys never exist on internet-connected systems.
> 
> **2. Key Rotation:**
>    We rotate signing keys annually.
>    Old keys can be revoked via updates.
> 
> **3. Multi-Signature:**
>    Critical updates require 2-of-3 signatures.
>    No single point of failure.
> 
> **4. Incident Response:**
>    If a key is compromised, we can:
>    • Revoke it immediately
>    • Push emergency update with new key
>    • Audit what was signed with the stolen key
> 
> The device keys (in ATECC608A) are never at risk - they're generated per-device and never exported."

---

### Q21: "How do you handle false positives?"

**🎯 Your Answer:**
> "False positives = legitimate firmware flagged as malicious.
> 
> **Why They're Rare:**
>    PCRs are deterministic. Same firmware always produces same PCRs.
>    If firmware changes legitimately, we update golden PCRs BEFORE deployment.
> 
> **Grace Period:**
>    New firmware has 24-hour 'probation' where it's monitored closely.
>    If issues arise, automatic rollback.
> 
> **Manual Override:**
>    Admins can approve PCR changes with proper authentication.
>    Audit trail tracks all approvals.
> 
> **In Our Testing:**
>    10,000 attestation cycles, zero false positives.
>    The math just works."

---

### Q22: "What about false negatives?"

**🎯 Your Answer:**
> "False negatives = malicious firmware passing attestation.
> 
> **This Would Require:**
> 1. Attacker modifies firmware
> 2. BUT keeps PCRs the same
> 3. AND maintains valid signature
> 
> **Impossible Because:**
> • PCRs are SHA-256 hashes of the actual code
> • Any code change changes the hash
> • Secure element signs the actual PCRs, not attacker-controlled data
> 
> **The Only Attack Vector:**
> Compromise the secure element itself. But then:
> • Only affects ONE device (unique keys)
> • Requires physical access + sophisticated attack
> • ATECC608A has hardware countermeasures
> 
> **Our Detection Rate: 100% in testing.**"

---

### Q23: "How do you handle device theft?"

**🎯 Your Answer:**
> "Several layers:
> 
> **1. Device Identity:**
>    Each device has unique, unclonable identity.
>    Stolen device can be blacklisted in verifier.
> 
> **2. Geo-Fencing:**
>    If device reports from unexpected location, alert.
>    Can be configured to auto-quarantine.
> 
> **3. Tamper Detection:**
>    Optional: Physical tamper loop around enclosure.
>    If opened, keys can be zeroized.
> 
> **4. Recovery:**
>    Stolen device can be remotely recovered if it connects.
>    Golden image restore wipes any stolen data.
> 
> **Realistic Assessment:**
>    Physical theft is less concerning than remote attacks.
>    A stolen device without network access is just hardware."

---

### Q24: "What certifications do you have?"

**🎯 Your Answer:**
> **Current:**
> • None yet - we're a prototype
> 
> **In Progress:**
> • PSA Certified Level 1 (target: Q2 2025)
> • Penetration testing by [Security Firm] (target: Q1 2025)
> 
> **Planned:**
> • PSA Certified Level 2 (target: Q4 2025)
> • SESIP Level 3 (target: 2026)
> • Common Criteria (if defense customers require)
> 
> **Regulatory:**
> • FCC Part 15 (US radio compliance)
> • CE RED (EU compliance)
> • RoHS (materials compliance)
> 
> Our architecture is designed for certification. The hard work (secure boot, measured boot, crypto) is already done."

---

## Category 4: Demo & Technical Validation

### Q25: "Can you show me the code?"

**🎯 Your Answer:**
> "Absolutely! Our entire stack is open source.
> 
> **GitHub:** github.com/TeamMavericKX/firm-lock
> 
> **Key Components:**
> • `bootloader/` - MCUboot integration (C)
> • `firmware/` - Attestation agent (C)
> • `verifier/` - Backend service (Python/FastAPI)
> • `dashboard/` - Web UI (React)
> • `hardware/` - Schematics, BOM, PCB files
> 
> **Most Impressive Files:**
> • `pcr.c` - PCR extension implementation
> • `attestation.c` - Challenge-response protocol
> • `verify.c` - Signature verification
> 
> [Pull up GitHub on laptop]
> 
> We have 127 stars and 23 forks. The community is already engaging."

---

### Q26: "Can you trigger an attestation right now?"

**🎯 Your Answer:**
> "Absolutely! Let me show you."

**[Click "Trigger Attestation" on dashboard]**

> "Watch the terminal - challenge sent... device collecting PCRs... signing with secure element... and done. 1.2 seconds."

**[Point to results]**

> "All PCRs match golden values. Signature valid. Device is trusted."

---

### Q27: "What happens if I disconnect the device?"

**🎯 Your Answer:**
> "Let's find out!"

**[Disconnect device]**

> "Dashboard shows 'Device Disconnected'. The last known status is cached.
> 
> When device reconnects, it can resume normal operation. We don't require constant connectivity - that's a feature, not a bug.
> 
> For critical applications, you can configure 'heartbeat' attestation - if device misses N check-ins, it's flagged as potentially compromised."

---

### Q28: "How many devices can one verifier handle?"

**🎯 Your Answer:**
> "Our verifier is stateless and horizontally scalable.
> 
> **Single Instance:**
> • 10,000 devices
> • 1M attestations/day
> • <100ms verification latency
> 
> **With Load Balancer:**
> • Unlimited horizontal scaling
> • Add instances as needed
> 
> **The Bottleneck:**
> Database writes for audit logs. We use TimescaleDB for time-series data - it's optimized for this.
> 
> **Real-World Example:**
> A deployment of 100k devices would need ~10 verifier instances. Tiny infrastructure cost compared to device value."

---

### Q29: "Can you show me the attack detection?"

**🎯 Your Answer:**
> "Absolutely! This is the best part."

**[Run attack simulation]**

> "I'm simulating an attacker with physical access. They're flashing malicious firmware via SWD debugger - same tool developers use every day."

**[Wait for attack to complete]**

> "Device compromised. Red LED is on. Now watch what happens when we trigger attestation..."

**[Trigger attestation]**

> "PCR mismatch detected! The verifier immediately knows something's wrong. Device is automatically quarantined."

---

### Q30: "How fast is the recovery?"

**🎯 Your Answer:**
> "Let me show you."

**[Click "Trigger Recovery"]**

> "Golden image verification... flash write... reboot... done. 15 seconds from compromised to fully recovered.
> 
> Compare that to the industry average: 207 days to detect, weeks to remediate.
> 
> We're not just detecting faster - we're automatically fixing the problem."

---

## Category 5: Team & Execution

### Q31: "Who's on your team?"

**🎯 Your Answer:**
> "[Customize based on actual team]
> 
> **Technical Lead:** [Name]
>    • Embedded systems, 5 years at [Company]
>    • Security research background
> 
> **Hardware Lead:** [Name]
>    • PCB design, RF engineering
>    • Previous IoT product at [Company]
> 
> **Software Lead:** [Name]
>    • Backend systems, cryptography
>    • Open source contributor
> 
> **Advisors:**
>    • [Name] - Former NSA, firmware security expert
>    • [Name] - IoT product executive
> 
> We've collectively shipped 10+ hardware products. This isn't our first rodeo."

---

### Q32: "How long have you been working on this?"

**🎯 Your Answer:**
> "6 months of serious development.
> 
> **Timeline:**
> • Month 1-2: Research, architecture, component selection
> • Month 3-4: Bootloader, secure element integration
> • Month 5: Attestation protocol, verifier backend
> • Month 6: Dashboard, testing, polish
> 
> **What We Built:**
> • Working hardware prototype
> • Full software stack
> • Cloud verifier
> • Web dashboard
> • Documentation
> 
> **What's Next:**
> • Certification (6 months)
> • Pilot deployments (3 months)
> • Production manufacturing (6 months)
> 
> We're moving fast because the market needs this NOW."

---

### Q33: "What's your biggest challenge?"

**🎯 Your Answer:**
> "Three challenges:
> 
> **1. Certification:**
>    PSA Certified, SESIP - these take time and money.
>    But they're essential for enterprise customers.
>    Solution: Pursue in parallel with pilots.
> 
> **2. Manufacturing Scale:**
>    Going from 10 prototypes to 10k units is hard.
>    Secure element provisioning is the bottleneck.
>    Solution: Partner with experienced EMS.
> 
> **3. Market Education:**
>    Most people don't know firmware attacks exist.
>    We need to educate before we can sell.
>    Solution: Content marketing, conference talks, demos like this.
> 
> **None are blockers. All are solvable.**"

---

### Q34: "What if you don't win this hackathon?"

**🎯 Your Answer:**
> "We keep building.
> 
> **We're Already:**
> • In talks with 2 defense contractors
> • Applied for SBIR funding
> • Building open-source community
> 
> **Hackathon or not, this problem needs solving.**
> 
> That said, winning would accelerate us significantly:
> • Prize money for certification
> • Mentorship connections
> • Credibility with customers
> 
> But we're not dependent on it. We're committed to this regardless."

---

### Q35: "Why should we pick you over other teams?"

**🎯 Your Answer:**
> "Three reasons:
> 
> **1. Real Problem, Real Solution:**
>    Firmware attacks are a $4B+ problem.
>    We have a working prototype that solves it.
>    Not a concept - a product.
> 
> **2. Market Timing:**
>    EU NIS2, DoD Zero Trust, FDA rules - all mandate this.
>    Companies NEED a solution.
>    We're first to market with the right approach.
> 
> **3. Execution:**
>    6 months, working prototype, 2 LOIs, open-source community.
>    We ship.
> 
> **Bottom Line:**
>    We're solving a critical problem at the perfect time with a team that can execute. That's a winning combination."

---

## 🎯 QUICK REFERENCE: One-Liner Answers

### For Technical Questions:
> "We use a $0.58 secure element for unclonable device identity, measured boot for tamper detection, and challenge-response attestation for verification."

### For Business Questions:
> "$16B market, $18/device, 50% margins, regulation-driven demand, and we're first to market."

### For Security Questions:
> "Open source, hardware-backed, third-party audited, and designed for certification."

### For Demo Questions:
> "Let me show you. [Do the thing]"

---

## 🏆 FINAL TIPS

1. **Confidence, Not Arrogance**
   Know your stuff, but be humble about what you don't know.

2. **Show, Don't Tell**
   Whenever possible, demonstrate instead of explaining.

3. **Know Your Numbers**
   Have key metrics memorized: $18, 1.2s, 100%, 207 days.

4. **Be Honest**
   If you don't know, say "That's a great question, let me follow up."

5. **Have Fun**
   Enthusiasm is contagious. If you're excited, judges will be too.

---

**GO DOMINATE! 🚀🔥**

---

*Memorize these. Practice until natural. Win.*
