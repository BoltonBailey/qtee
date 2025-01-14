:::info
:construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction:

_This document is just a draft. Not to be taken too seriously._ :slightly_smiling_face: 

:construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction:
:::

# qTEE: Moving Towards Open Source and Verifiable Secure-through-Physics TEE Chips
This is an initiative to spark research to explore how we could develop a secure chip for TEEs (Trusted Execution Environments) that would ultimately be secure because of physics rather than economics[^1]. The chip design should be open source, and its physical implementation should be verifiable, meaning that it should match the open source design. Moreover, the root of trust (embedded secret key) should be proven to have not leaked during generation or manufacturing. Thus, the hope and vision is to develop a TEE chip that does not need to be trusted because it can be verified by physics and mathematics. For an example of a cryptographic protocol implementation that is secure through physics see [Experimental relativistic zero-knowledge proofs] by _Alikhani et al_.


:::danger 
To put this vision into context, current TEEs such as Intel SGX, face the following challenges:

1. :radioactive_sign: **NO proof of manufacturing** according to a known open source chip design specification
2. :radioactive_sign: **NO proof of non-leakage of secret bits** -- how can we know that the secret bits (root of trust) encoded into the chip were not leaked during manufacturing
3. :radioactive_sign: **NO proof of hidden-forever secret bits** -- above and beyond trusting or not trusting the chip manufacturers, and the manufacturing processes, one problem remains: Can we truly hide secret bits of information (root of trust) into physical matter?
4. :radioactive_sign: **Centralized remote attestation** -- meaning that trust in the manufacturer is required to attest the trustworthiness of a TEE. [[RFC 9334]]

See https://github.com/sbellem/qtee/issues/2, for more details[^2].
:::

## Overview
The key topics that this document wishes to explore are:

* [Revisiting the Problem which TEEs aim to solve](#The-Problem-TEEs-aim-to-solve)
* [Do we really need TEEs? Could we do it all with mathematics (FHE, ZKP, MPC, etc)?](#Do-we-really-need-TEEs?)
* [Motivations for better TEEs](#Motivation)
    * [Don't Trust, Verify ... Or use TEEs?](#Dont-Trust-Verify-…-Or-use-TEEs)
    * [Kerckhoffs's Principle applied to Chip Design](#Kerckhoffss-Principle-applied-to-Chip-Design)
    * [Related Work](#Related-Work)
* [Threat Model](#Threat-Model)
* [Cypherpunk-Friendly Chip](#Cypherpunk-Friendly-Chip)
    * [Verifiable Chip based on an Open Source Hardware Design](#Verifiable-Chip-based-on-an-Open-Source-Hardware-Design)
    * [Marching Towards DAMOs (aka Zero Trust Manufacturing)](#Marching-Towards-DAMOs)
    * [Root of Trust with PUFs](#Root-of-Trust-with-PUFs)
    * [Decentralized Remote Attestation](#Decentralized-Remote-Attestation)
* [Beyond PUFs: Cryptography and Physics United](#Beyond-PUFs-Cryptography-and-Physics-United)
* [Appendix: Intel SGX's Root of Trust](#Appendix-Intel-SGXs-Root-of-Trust)
* [Appendix: Chip Attacks -- What does it take?](#Appendix-Chip-Attacks-–-What-does-it-take?)


## The Problem TEEs aim to solve
TEEs are an attempt to solve the _secure remote computation_ problem. Quoting [Intel SGX Explained] by _Victor Costan and Srinivas Devadas_:

:::info
> _Secure remote computation is the problem of executing software on a remote computer owned and maintained by an untrusted party, with some integrity and confidentiality guarantees._
:::

Note that the remote computer is said to be owned and maintained by an _untrusted_ party. Yet, current TEEs, cannot handle physical attacks such as chip attacks (see [TEE Chip Attacks: What does it take?](#Appendix-Chip-Attacks-–-What-does-it-take?)), which would allow an attacker to retrieve the root of trust (secret keys encoded in the hardware). Once an attacker knows the secret keys, it can emulate a TEE, and go through the attestation process unnoticed (e.g. see Appendix A. Emulated Guard eXtensions in https://sgx.fail/ paper).

Is it even possible to build a chip that can handle physical attacks, such as those making use of Focus Ion Beam microscopes as mentioned in [Intel SGX Explained] (section 3.4.3)? One could argue that it's not possible in the classical setting, but may be possible in the quantum setting. Some argue that PUFs (Physical Unclonable Functions) cannot be broken and would therefore be a solution. However, there's plenty of research that focuses of breaking PUFs, and there's also active research in developping more secure PUFs. Hence, it seems reasonable to assume that PUFs are not an ultimate solution to chip attacks, although they do seem to be a major improvement. (See [Root of Trust with PUFs](#Root-of-Trust-with-PUFs).)

## Do we really need TEEs?
**Why can't we do it all with FHE, ZKP, and MPC?**

Not sure. :smile: Besides the performance limitations of FHE, ZKP and MPC, the problem of proof-of-deletion or certified deletion may be the most mentioned one. The intuition is simple: "How do you prove that you completely forgot what some secret data was deleted?" You could show that your harddisk has been completely wiped out, but perhaps you copied it elsewhere. Hence, certified deletion appears to not be possible in the classical setting but it apparently is if one is willing to step one foot (or two), into the quantum setting (e.g.: [High-Dimensional Quantum Certified Deletion] by _Hufnagel et al_, [Quantum Proofs of Deletion for Learning with Errors] by _Poremba_). If we are confined to the classical setting though, then TEEs may be useful. If the program generating and/or handling secrets is executed in a TEE then the program can be written such that it will delete the secrets once it's done with the task. As an alternative to TEEs, there's the idea of traceable secret sharing as presented in [Traceable Secret Sharing: Strong Security and Efficient Constructions] by _Boneh et al_.

## Motivation
According to [SoK: Hardware-supported TEEs] and [Intel SGX Explained], current chips that implement TEEs cannot protect against physical attacks such as chip delayering, which would allow an attacker to extract the so-called root of trust, meaning hardware embedded secret keys upon which the entire security of the TEE depends. The only current known defense against chip attacks is trying to make the cost of a chip attack as high as possible. To make things worst, it's not even clear what the cost of a chip attack is; perhaps one million dollar (see [TEE Chip Attacks: What does it take?](#Appendix-Chip-Attacks-–-What-does-it-take?))? So, at the very least, one would hope we would know what the cost of a chip attack is, such that protocol designers could [design mechanisms][mechanism design] that would eliminate economic incentives to attack the chip, because the cost of the attack would not be worth what could be extracted out of the attack. It's very important to note here that a protocol relying on TEEs may also be targeted for attacks for reasons other than financial, and it's probably best to avoid using TEEs for such cases (e.g. privacy preserving application used by political dissidents).

Aside from being vulnerable to chip attacks the current popular TEEs, such as Intel SGX, are closed source, meaning that their hardware designs are not public, which in turn makes it very difficult to know whether a chip is implemented as claimed. Even with an open source hardware design we would need to figure out how to verify that the chip was implemented as per the open source design, and that secrets generated and embedded into the hardware at the time of manufacturing were not leaked.


### Don't Trust, Verify ... Or use TEEs?
In the crypto world, the motto "Don't Trust, Verify" is frequently used to emphasize the verifiability feature of the various protocols, which allows any user to verify for themselves the validity of a transaction or claim. It may be said that the backbone of the reverred verifiability is cryptography and distributed systems, which involves trusting mathematics and trusting an honest majority, respectively. Consensus protocols, and many multi-party computation (MPC) protocols require to trust that the majority of the validators are honest. The majority may range from 51% to 75% depending on the protocol. So on one hand the world of crypto is secured through a combination of mathematics and trust in an "honest majority". So what about TEEs? Where do they fit in this picture?

The so-called web3 world (aka as crypto space) increasingly makes use of TEEs (mostly Intel SGX) in applications where substantial amounts of money may flow, and where TEEs help secure the confidentiality of its users. It's therefore important to properly understand what it means to trust TEEs. For a strange reason, it seems complicated to answer the question of "What does it mean to trust TEEs?" If you ask different people, you may find a spectrum of different answers ranging from the likes of: "You have to trust the chip maker! But you already trust them anyways." to "Intel SGX is broken every month, I don't understand why people use them!"

:::warning
In general, it may be fair to say that trusting a TEE means the following:

1. Trust that the chip is **designed** as per the claims of the chip maker.
2. Trust that the chip is **manufactured** as per the claims of the chip maker.
3. Trust that the **root of trust** is not leaked during the manufacturing process.
4. Trust that the **root of trust** cannot be extracted out "cheaply" or "easily" by an attacker who has physical access to the chip.
5. Trust the **remote attestation** process, which may mean having to trust the role of the manufacturer (e.g. Intel SGX with EPID or DCAP).[^3]

Note that the above implicitly assumes that the design and implementation are secure, free of bugs.[^4]
:::

### Kerckhoffs's Principle applied to Chip Design
:::success
[Auguste Kerckhoffs](https://en.wikipedia.org/wiki/Auguste_Kerckhoffs), back in 1883, in his paper entitled [La Cryptographie Militaire](https://www.petitcolas.net/kerckhoffs/la_cryptographie_militaire_i.htm) (Military Cryptography), argued that security through obscurity wasn't a desirable defense technique.

> **_Il faut qu’il n'exige pas le secret, et qu'il puisse sans inconvénient tomber entre les mains de l’ennemi_**

roughly translated to:

> **_It must not require secrecy, and it must be capable without inconvenience to fall into the enemy's hand_**

(Perhaps, one may point out that Kerckhoffs was assuming that the private key would be held secretly and not be part of an open design. The need to secure a private key in an open design begs for physics to enter the arena(e.g. PUFs).

For example, the Secure Cryptographic Implementation Association (SIMPLE-Crypto Association) aims to apply 
Kerckhoffs's Principle to hardware and lays out their vision at https://www.simple-crypto.org/about/vision/:

> **[...] our vision is that as research advances, the security by obscurity paradigm becomes less justified and its benefits are outweighted by its drawbacks.** That is, while a closed source approach can limit the adversary's understanding of the target implementations as long as their specifications remain opaque, it also limits the public understanding of the mechanims on which security relies, and therefore the possibility to optimize them. By contrast, an open approach to security can lead to a better evaluation of the worst-case security level that is targeted by cryptographic designs.
:::

:::danger
For some reason, the hardware world does not embrace open source like the software world. Moreover, it is common practice to use [security through obscurity](https://en.wikipedia.org/wiki/Security_through_obscurity) as a core design principle to secure hardware. Simply said, for whatever reason, the current hardware industry appears to be dominated by the belief that it's best to hide the design and inner workings of a chip by adding unnecessary elements to the design, just to confuse a potential attacker, in the hope that the attacker will not be able to understand the design, and thus reverse engineer it.
:::


### Related Work
[Lessons Learned from Blockchain Applications of Trusted Execution Environments and Implications for Future Research](https://arxiv.org/pdf/2203.12724)


## Threat Model
**The worst.**

* Attackers with physical access to the chip, with unlimited resources and funds **MUST** be considered
* State actors
* Malicious actors with full access to every step of the supply chain, foundries, etc
* Malicious actors with unlimited access to data centers, e.g. swapping computers with their own fake SGX computers without being noticed
* etc, etc.

**Just think the worst of the worst.**

_Perhaps_ the only thing that may be out-of-bound is remote civilizations or state actors with access to new physics that is not yet known by the general public (e.g. academia/universities). For instance, imagine another planet where beings would know how to go faster than the speed of light.

### Relevant Readings
:::danger
[The battle for Ring Zero] _by Cory Doctorow_.
> _But how can we trust those sealed, low-level controllers? What if manufacturers – like, say, Microsoft, a convicted criminal monopolist – decides to use its low-level controllers to block free and open OSes that compete with it? What if a government secretly (or openly) orders a company to block privacy tools so that it can spy on its population? What if the designers of the secure co-processor make a mistake that allows criminals to hijack our devices and run code on them that, by design, we cannot detect, inspect, or terminate?_
>
> _That is: to make our computers secure, we install a cop-chip that determines what programs we can run and stop. To keep bad guys from bypassing the cop-chip, we design our computer so it can't see what the cop-chip is doing. **So what happens if the cop-chip is turned on us?**_
:::


## Cypherpunk-Friendly Chip
As mentioned in [The Problem TEEs aim to solve](#The-Problem-TEEs-aim-to-solve), if the problem we wish to tackle is that of secure remote computation, the threat model should include attackers with physical access to the chip, which means that the chip should be secure against physical attacks, which begs the question as to whether this is even possible in the classical setting (i.e. without using quantum physics). That being said, it does not mean that we cannot improve the current TEEs. This section aims to explore what we could feasibly do today to have a chip that attempts to align itself with the motto of "Don't Trust, Verify", omnipresent in the web3 and cypherpunk cultures.

In the context of a secure chip, the motto **"Don't Trust, Verify"** calls for at least four fundamental pillars, which address the challenges presented at the beginning of this document:

:::success
1. **Proof of manufacturing** according to a known open source chip design specification
2. **Proof of non-leakage of secret bits** to verify that the root of trust wasn't leaked during manufacturing
3. **Proof of hidden-forever secret bits** -- the root of trust must be proven to be unbreakable
4. **Decentralized Remote Attestation** -- :grin: a device should be able to provide a proof that it is what it claims to be.
:::



### Verifiable Chip based on an Open Source Hardware Design
Having an open source hardware design is perhaps the most reasonable place to start. Verifying that a physical chip does implement the intended open source hardware design is perhaps more difficult, and we can try to tackle this in a second step. Hence, we'll first start by exploring how we could have a TEE chip with an open source hardware design.

#### Open Source Hardware Design
This is not a new idea, and it may be useful to survey current and past efforts such as:

* [Tiny Tapeout]
* [Chips Alliance]
* [Caliptra]
* [Google Titan]
* [LibreSilicon]
* [The Silicon Salon]
* [Free Silicon Conference 2024](https://wiki.f-si.org/index.php/FSiC2024)

##### Resources
[Tiny Tapeout] has a lot of educational material at that may be worth reading for those who don't have a background in hardware. 
 
Also worth having a look at is the course [Zero to ASIC Course].


#### Verfiable Chip Implementation
This is also not a new problem. One approach is to use [Logic Encryption], which somehow locks the chip design to protect against a  malicious foundry. The company [HENSOLDT Cyber] has numerous research works on the topic, in addition to actually making chips, and hence, is probably worth studying. Their papers are listed at https://hensoldt-cyber.com/scientific-papers/, but let's list a few here:

* [Scaling Logic Locking Schemes to Multi-Module Hardware Designs](https://www.ice.rwth-aachen.de/publications/publication/sisejkovicARCS2020/)
* [Inter-Lock: Logic Encryption for Processor Cores Beyond Module Boundaries](https://www.ice.rwth-aachen.de/publications/publication/sisejkovicETS2019/)
* [A Critical Evaluation of the Paradigm Shift in the Design of Logic Encryption Algorithms](https://www.ice.rwth-aachen.de/publications/publication/sisejkovicVLSIDAT2019/)
* [A Unifying Logic Encryption Security Metric](https://www.ice.rwth-aachen.de/publications/publication/sisejkovicSAMOS2018/)


 
#### GitHub Issues
* https://github.com/sbellem/qtee/issues/1
* https://github.com/sbellem/qtee/issues/2#issuecomment-1648191994



### Marching Towards DAMOs
**DAMO: Decentralized Autonomous Manufacturing Organization**

How can we be certain that the manufacturing process did not leak the secret keys (root of trust)? Could the supply chain somehow produce a proof of non-leakage of secret keys?

Could we somehow bootstrap a fully automated foundry, where the manufacturing process is fully programmed, verifiable, and chips are built atom-by-atom.

Or, could we build chips at home?


#### Zero Trust Manufacturing
_Can we learn something interesting from Zero Trust applied to Chip Manufacturing?_

* [Zero trust security model](https://en.wikipedia.org/wiki/Zero_trust_security_model)
* [Intel: A Zero Trust Approach to Architecting Silicon](https://www.intel.com/content/www/us/en/newsroom/opinion/zero-trust-approach-architecting-silicon.html#gs.43wv53)
* [Chip Industry Needs More Trust, Not Zero Trust](https://semiengineering.com/chip-industry-needs-more-trust-not-zero-trust/)
* [Building a Zero Trust Security Model for Autonomous Systems ](https://spectrum.ieee.org/zero-trust-security-autonomous-systems) (See "Zero Trust Applied to Chip Design" section)
* [Zero Trust Security In Chip Manufacturing](https://youtu.be/OsjMcjGkgVE?si=G0nInzcmRRrXhaSg)

#### Building chips atoms by atoms?
_Nanofactories, nanomanufacturing, atomically precise manufacturing, etc._
* [Nanofactory](https://www.molecularassembler.com/Nanofactory/index.htm)
* [Productive Nanosystems](https://en.wikipedia.org/wiki/Productive_nanosystems)
* [Nanosystems: Molecular Machinery, Manufacturing, and Computation](https://web.archive.org/web/20191008162657/http://e-drexler.com/d/06/00/Nanosystems/toc.html) _by Eric Drexler_
* [An Introduction to Molecular Nanotechnology](https://youtu.be/cdKyf8fsH6w?si=bE-kHxiHpvKj8Wq3) _with Ralph Merkle_
* [Molecularly Precise Fabrication and Massively Parallel Assembly: The Two Keys to 21st Century Manufacturing](https://www.molecularassembler.com/Nanofactory/TwoKeys.htm) _by Robert A. Freitas Jr. and Ralph C. Merkle_
* [Engines of Creation 2.0, The Coming Era of Nanotechnology](https://web.archive.org/web/20140810022659/http://www1.appstate.edu/dept/physics/nanotech/EnginesofCreation2_8803267.pdf) _by Eric Drexler_

#### GitHub Issue
https://github.com/sbellem/qtee/issues/7


### Root of Trust with PUFs
[Physical Unclonable Functions](https://en.wikipedia.org/wiki/Physical_unclonable_function) are arguably the current best hope to protect against physical attacks aimed at extracting secret keys (root of trust). That being said, PUFs are an active area of research where new PUFs design are proposed and existing designs are broken. Hence, research is needed to better understand the limitations of PUFs in the context of TEEs.

Not sure where it's best to start, but perhaps this article (if you have access):  
[Physical unclonable functions](https://www.nature.com/articles/s41928-020-0372-5) by [Yansong Gao](https://www.nature.com/articles/s41928-020-0372-5#auth-Yansong-Gao-Aff1-Aff2), [Said F. Al-Sarawi](https://www.nature.com/articles/s41928-020-0372-5#auth-Said_F_-Al_Sarawi-Aff3) & [Derek Abbott](https://www.nature.com/articles/s41928-020-0372-5#auth-Derek-Abbott-Aff4)

OR:

* [Physical Unclonable Functions for Device Authentication and Secret Key Generation](https://people.csail.mit.edu/devadas/pubs/puf-dac07.pdf)

  > Because the PUF circuit is rather simple, attackers can try to construct a precise timing model and learn the parameters from many input-output pairs [8]. To prevent these model-building attacks, the PUF circuit output can be obfuscated by XOR’ing multiple outputs or a PUF output can be used as one of the MUX control signals. **Note that the model building attack is irrelevant for the cryptographic key generation where the PUF output is never directly exposed.** [G. Edward Suh, Srinivas Devadas](https://people.csail.mit.edu/devadas/pubs/puf-dac07.pdf)

* [An Introduction to Physically Unclonable Functions](https://www.allaboutcircuits.com/technical-articles/an-introduction-to-physically-unclonable-functions/)

  > When manufactured, the PUF will be fed a series of different challenges and have its responses recorded. Through this exercise, the designers know each PUF's unique response to a given challenge and can use this information to prevent counterfeiting, create and store cryptographic keys, and many other security feats.
  
  TODO: figure out if the set of CRPs is not needed for signing keys. Also, out of curiosity could there be oblivious (or zk) CRPs, meaning that no one knows the challenge response pairs, but yet, they can be used.
  

#### First well-known PUF: Physical One-Way Functions
https://nbviewer.org/github/rpappu/pdf-publications/blob/master/Pappu-Science-2002.pdf

#### Remote Attestation
* [A lightweight remote attestation using PUFs and hash-based signatures for low-end IoT devices](https://www.sciencedirect.com/science/article/pii/S0167739X23002236)
* [SMART: Secure and Minimal Architecture for (Establishing a Dynamic) Root of Trust](https://ics.uci.edu/~gts/paps/smart.pdf)

#### Malicious PUFs
* [Feasibility and Infeasibility of Secure Computation with Malicious PUFs](https://eprint.iacr.org/2015/405)
* [On the Security of PUF Protocols under Bad PUFs and PUFs-inside-PUFs Attacks](https://eprint.iacr.org/2016/322)
* [Everlasting UC Commitments from Fully Malicious PUFs](https://eprint.iacr.org/2021/248)

#### New PUFs
* https://arxiv.org/abs/2310.19587
* https://pubs.aip.org/aip/sci/article/2019/29/290009/360043/Fingerprinting-silicon-chips-just-got-easier
* [Spectral sensitivity near exceptional points as a resource for hardware encryption](https://www.nature.com/articles/s41467-023-36508-x)

#### Applications
##### [PUF-derived IoT identities in a zero-knowledge protocol for blockchain](https://www.sciencedirect.com/science/article/abs/pii/S2542660518301124)
> In this paper, an alternative authentication approach in which an MCU generates a secret key internally is introduced, exploiting manufacturing variability as a physical unclonable function (PUF). As the key is generated by the device itself, manufacturers save the expense of a secure environment for external key generation. In production, once chips are loaded with a firmware, it is only necessary to run an internal characterization and pass on the resulting public key, mask and helper data to be stored for authentication and recovery. Further external memory access is prevented, e.g., by blowing the JTAG security fuse. As the secret key is regenerated (with the same result each time) rather than stored in non-volatile memory, it is very hard to clone and the cost of a secure element can be saved.

> The case for such IoT devices is strengthened further in combination with a distributed ledger, or blockchain. First of all, the immutability and distributed trust provided by a blockchain can make the device authentication independent of the manufacturer. Secondly, a business process implemented in chaincode that relies on IoT inputs can validate device signatures to ensure the authenticity and integrity of those inputs.

> Replacing the central database operated by a manufacturer with a blockchain makes the system independent of the manufacturer. The chaincode will still allow only the manufacturer to create new machine entries on the distributed ledger but as the ledger content is distributed to all participants (multiple manufacturers, retailers, owners, etc.) the manufacturer is relieved of administering the system and guaranteeing its availability. A central database would go offline when the manufacturer goes out of business whereas a blockchain can survive.
> 
> Given the security disadvantages of symmetric authentication schemes (keeping a database of keys to authenticate with the risk of being hacked or lost, the risk of cloning, and barriers for third-party authentication, among others) our approach instead uses public-key cryptography based on learning parity with noise (LPN) problems, and in particular zero-knowledge (ZK) protocols to further simplify the management of device public keys. The blockchain may make the public keys generated by each device available for anyone to use in their own authentication system.
> 
> As for the second aspect, even a low-cost device can prevent manipulation of its communication with a blockchain by signing its messages with our PUF-derived keys, making the proposal suitable for any resources-limited device connected to the blockchain [9]. The chain code, in turn, can also validate the device signatures to ensure data integrity and authenticity, extending the trust the blockchain provides into the IoT device.
> 
> This paper proposes using an SRAM-based PUF to generate cryptographic keys that are employed in a zero-knowledge proof to authenticate an IoT device. We present an efficient implementation in an MCU and show that even low-cost devices can perform the required computational tasks sufficiently fast. Experimental results demonstrate that our approach is robust against temperature variations and that collisions of device identities are unlikely.

##### [A survey on physical unclonable function (PUF)-based security solutions for Internet of Things](https://www.sciencedirect.com/science/article/pii/S1389128620312275)

#### Commercial PUFs
https://www.cryptoquantique.com/products/qdid/

#### Concerns/Questions
As per [Physical unclonable functions](https://www.nature.com/articles/s41928-020-0372-5):

> Authentication can also be executed remotely, once the CRP (challenge–response pair) is recorded in a secure database only known by the trusted party (server).

This seems to be relating to what is called remote attestation in the context of popular TEEs like SGX. In the context of SGX, for instance, the chip manufacturer is considered to be a trusted party, for various reasons (e.g: https://github.com/sbellem/qtee/issues/2).

#### Hacking & Cryptanalysis
* https://github.com/nils-wisiol/pypuf (cryptanalysis)
* https://asvin.io/physically-unclonable-function-setup/
* https://github.com/nils-wisiol/LP-PUF
* https://github.com/stnolting/fpga_puf
* https://www.crypto.ruhr-uni-bochum.de/imperia/md/crypto/kiltz/ulrich_paper_47.pdf

#### Specifications in Chip Designs
* https://github.com/chipsalliance/Caliptra/blob/main/doc/Caliptra.md#future-effort-caliptra-security-subsystem
* https://github.com/chipsalliance/caliptra-rtl/blob/main/docs/CaliptraIntegrationSpecification.md

#### References
* [Physical Unclonable Functions for Device Authentication and Secret Key Generation](https://people.csail.mit.edu/devadas/pubs/puf-dac07.pdf)
* [Feasibility and Infeasibility of Secure Computation with Malicious PUFs](https://eprint.iacr.org/2015/405)
* [Providing Root of Trust for ARM TrustZone using On-Chip SRAM](https://eprint.iacr.org/2014/464)
* [Making sense of PUFs](https://semiengineering.com/pufs-promise-better-security/)

### Decentralized Remote Attestation
:construction: TODO :construction: 

Not sure how this could be achieved. Conceptually speaking, a device should be able to proof what it claims to be with respect to both its hardware and software, without relying on a trusted third party such as the manufacturer. [RFC 9334 - Remote ATtestation procedureS (RATS) Architecture](https://rfc-editor.org/rfc/rfc9334.html) may be useful to review in the context of our [threat model](#Threat-Model).

For instance, in the case of Intel SGX, the chip manufacturer plays a central role in the remote attestation process. It may be useful to go through all the steps that Intel plays (e.g. provisioning attestation keys, verifying quotes, etc) and think through to see how these steps could be decentralized. Perhaps first defining the [ideal functionality] for remote attestation would be useful; or reviewing works that have already done so. Once we have the ideal functionality defined

#### Ideal Functionality for Remote Attestation
:construction: TODO :construction: 
See perhaps [Cryptographically Assured Information Flow: Assured Remote Execution](https://arxiv.org/abs/2402.02630) for inspiration.


#### Thought experiment

:::success
:construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction:
:construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction:
:construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction:
 
#### :thought_balloon: Thought experiment
If somehow we have managed to manufacture a chip, in a "decentralized" way, such that it can be verified, then perhaps the "decentralized" manufacturing process could log public metadata about the chip that would uniquely identify it. For instance, the metadata could be tied to a fingerprint generated via a PUF, in the chip. The metadata would contain the proof of correct manufacturing with respect to the requirements discussed earlier, such as matching a (formally verified) open source hardware design, and not leaking secret bits.

Then remote attestation in this case would involve first requesting from the device that it provides its unique fingerpring which could then be verified against the public metadata ... but how could we prevent devices from providing a fake fingerprint? Perhaps the public records of correctly manufactured devices should not be public afterall. That is, a chip's fingerprint should not be publicly linkable to the metadata (proofs of correct manufacturing). Said differently, a verifier should just needs to know that the chip it is interacting with has been manufactured correctly, and the verification process should not reveal information that could be used by a malicious chip to forge a fake identity.

We also need a proof that it loaded the expected software for execution ...

:construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction:
:construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction:
:construction: :construction: :construction: :construction: :construction: :construction: :construction: :construction: 
:::

#### Readings
* [Cryptographically Assured Information Flow: Assured Remote Execution](https://arxiv.org/abs/2402.02630)
* https://web.cs.wpi.edu/~guttman/pubs/good_attest.pdf
* https://arxiv.org/abs/2105.02466
* https://seclab.stanford.edu/pcl/cs259/projects/cs259_final_lavina_jayesh/CS259_report_lavina_jayesh.pdf
* https://github.com/ietf-rats-wg/architecture?tab=readme-ov-file
* https://link.springer.com/article/10.1007/s10207-011-0124-7
* https://arxiv.org/pdf/2308.11921
* https://arxiv.org/pdf/2306.14882
* https://arxiv.org/pdf/2204.06790


## Beyond PUFs: Cryptography and Physics United
_See https://github.com/sbellem/qtee_

Since a TEE is ultimately a physical device, in which secret bits are embedded, it seems inevitable that soon or later we'll have to confront the question of whether it's really physically possible to hide these secret bits. Current efforts and hopes appear to rest on economic incentives at best, meaning that the costs of breaking into the physical device are hoped to be too high for the gains that the attacker will get in return.  But what if we could design and implement chips that are secure as long as the physics is not broken. That is, chips for which breaking their security would mean breaking laws of physics. This is not a new concept, and has been done in [Physical One-Way Functions] _by Ravikanth Pappu_ and [Experimental relativistic zero-knowledge proofs] _by Alikhani et al._ for instance. 


### Equivalence Relations between High Energy Physics and Cryptography
[Black-Hole Radiation Decoding is Quantum Cryptography](https://arxiv.org/abs/2211.05491)

### Trusted Black Hole Execution Environments
[Black Hole Computers](https://www.scientificamerican.com/article/black-hole-computers-2007-04/)



## Appendix: Intel SGX's Root of Trust
If we take Intel as an example, trusting the chip manufacturer means many things. Intel SGX's so-called root of trust rests on two secret keys (seal secret and provisionaing secret), and an attestation key, as shown in the figure below, from [Intel SGX Explained]. Note that this may have changed since the writing of the [Intel SGX Explained] paper, but at the time at least, the two secrets were said to be stored in e-fuses inside the processor's die. Moreover, the two secret keys, stored in e-fuses, were encrypted with a global wrapping logic key (GWK). The GWK is a 128-bit AES key that is hard-coded in the processor's circuitry, and serves to increase the cost of extracting the keys from an SGX-enabled processor. The Provisioning Secret was said to be generated at the key generation facility - burned into the processor's e-fuses and stored in Intel's Provisioning Service DB. The Seal Secret was said to be generated inside the processor chip, and claimed not to be known to Intel. Hence, trusting Intel meant to trust that they do not leak the attestation key, and the provisioning key as they have access to them. Trusting Intel also meant that the manufacturing process that generates and embeds the Seal Secret did not leak the secret key. Trusting Intel also meant that once a chip is made, they did not attempt to extract the Seal Key, which is the only key, out of three, which they did not know.


![image](https://hackmd.io/_uploads/rydXhPCTa.png)

## Appendix: Chip Attacks -- What does it take?
:::danger
**tl;dr**: Chip attacks cannot be prevented, but only made expansive to carry on, which is very relative, depending on the application in which the chip is used. Furthermore, as far as I know, there's no known chip attack that has been reported along with its required cost. Hence, currently we can only speculate that an attack may be in the range of a 1 million dollars, judging from the cost of focused ion beam (FIB) microscopes and guessing how much money a team of experts would cost. In the context of crypto/web3, protocol designers should probably be extremely careful, given that many protocols move massive amounts of money; in the hundreds of millions, and more.
:::
:::success
It would be extremely useful to see actual chip attacks being reported by research groups, as it would help to set a price on such attacks, and the price of the attack could be used by protocol designers.
:::

---

By chip attacks here, we mean those described in [Intel SGX Explained], _section 3.4.3_. The paper is from 2016, and at the time of writing the authors wrote that the Intel's CPU had a [feature size](https://en.wikipedia.org/wiki/Semiconductor_device_fabrication#Feature_size) of 14nm. In the interest of being pro-active to understand current or future chips, we could assume [3nm](https://en.wikipedia.org/wiki/3_nm_process) feature size perhaps. But it's not clear what this exactly means, because apparently these numbers are more of a marketing act, as per [3 nm process](https://en.wikipedia.org/wiki/3_nm_process):

> The term "3 nanometer" has no direct relation to any actual physical feature (such as gate length, metal pitch or gate pitch) of the transistors. According to the projections contained in the 2021 update of the [International Roadmap for Devices and Systems](https://en.wikipedia.org/wiki/International_Roadmap_for_Devices_and_Systems) published by IEEE Standards Association Industry Connection, a "3 nm" node is expected to have a contacted gate pitch of 48 nanometers, and a tightest metal pitch of 24 nanometers.[[12]](https://en.wikipedia.org/wiki/3_nm_process#cite_note-IRDS-12)
>
> However, in real world commercial practice, "3 nm" is used primarily as a marketing term by individual microchip manufacturers (foundries) to refer to a new, improved generation of silicon semiconductor chips in terms of increased transistor density (i.e. a higher degree of miniaturization), increased speed and reduced power consumption.[[13]](https://en.wikipedia.org/wiki/3_nm_process#cite_note-13)[[14]](https://en.wikipedia.org/wiki/3_nm_process#cite_note-14) There is no industry-wide agreement among different manufacturers about what numbers would define a "3 nm" node.[[15]](https://en.wikipedia.org/wiki/3_nm_process#cite_note-IRDS2-15)

In any case, it's quite clear that instrumentation to work at a smaller scale is needed.

Back to [Intel SGX Explained](https://eprint.iacr.org/2016/086), _section 3.4.3_, some key excerpts:

> The most equipment-intensive physical attacks involve removing a chip’s packaging and directly interacting with its electrical circuits. These attacks generally take advantage of equipment and techniques that were originally developed to diagnose design and manufacturing defects in chips. [[22]] covers these techniques in depth.
>
>The cost of chip attacks is dominated by the required equipment, although the reverse-engineering involved is also non-trivial. This cost grows very rapidly as the circuit components shrink. At the time of this writing, the latest Intel CPUs have a 14nm feature size, which requires ion beam microscopy.
>
> The least expensive classes of chip attacks are destructive, and only require imaging the chip’s circuitry. These attacks rely on a microscope capable of capturing the necessary details in each layer, and equipment for mechanically removing each layer and exposing the layer below it to the microscope.
>
>  E-fuses and polyfuses are particularly vulnerable to imaging attacks, because of their relatively large sizes.
>
> [...], once an attacker develops a process for accessing a module without destroying the chip's circuitry, the attacker can use the same process for both passive and active attacks.
>
> **At the architectural level, we cannot address physical attacks against the CPU’s chip package.** [...]
>
> Thankfully, **physical attacks can be deterred by reducing the value that an attacker obtains by compromising an individual chip. As long as this value is below the cost of carrying out the physical attack, a system's designer can hope that the processor's chip package will not be targeted by the physical attacks.**
>
> Architects can reduce the value of compromising an individual system by avoiding shared secrets, such as global encryption keys. Chip designers can increase the cost of a physical attack by not storing a platform's secrets in hardware that is vulnerable to destructive attacks, such as e-fuses.

> [[22]]: Friedrich Beck. _Integrated Circuit Failure Analysis: a Guide to Preparation Techniques._ John Wiley & Sons, 1998.

There's also a brief discussion of PUFs in a security analysis section of [Intel SGX Explained], section _6.6.2 Physical Attacks_:

> The threat model stated by the SGX design excludes physical attacks targeting the CPU chip (§ 3.4.3).  Fortunately, Intel’s patents disclose an array of countermeasures aimed at increasing the cost of chip attacks.
>
> For example, the original SGX patents [110, 138] disclose that the Fused Seal Key and the Provisioning Key, which are stored in e-fuses (§ 5.8.2), are encrypted with a global wrapping logic key (GWK). The GWK is a 128-bit AES key that is hard-coded in the processor’s circuitry, and serves to increase the cost of extracting the keys from an SGX-enabled processor.
>
> As explained in § 3.4.3, e-fuses have a large feature size, which makes them relatively easy to “read” using a high-resolution microscope. In comparison, the circuitry on the latest Intel processors has a significantly smaller feature size, and is more difficult to reverse engineer. **Unfortunately, the GWK is shared among all the chip dies created from the same mask, so it has all the drawbacks of global secrets explained in § 3.4.3.**
>
> Newer Intel patents [67, 68] describe SGX-enabled processors that employ a Physical Unclonable Function (PUF), e.g., [175], [133], which generates a symmetric key that is used during the provisioning process.
>
> Specifically, at an early provisioning stage, the PUF key is encrypted with the GWK and transmitted to the key generation server. At a later stage, the key generation server encrypts the key material that will be burned into the processor chip’s e-fuses with the PUF key, and transmits the encrypted material to the chip. The PUF key increases the cost of obtaining a chip’s fuse key material, as an attacker must compromise both provisioning stages in order to be able to decrypt the fuse key material.
>
> As mentioned in previous sections, patents reveal design possibilities considered by the SGX engineers. However, due to the length of timelines involved in patent applications, patents necessarily describe earlier versions of the SGX implementation plans, which might not match the shipping implementation. We expect this might be the case with the PUF provisioning patents, as it makes little sense to include a PUF in a chip die and rely on e-fuses and a GWK to store SGX’s root keys. Deriving the root keys from the PUF would be more resilient to chip imaging attacks.

I don't know whether the latest Intel SGX chips make use of PUFs, and how accurate the above still is for the latest chips.

:::danger
In any case, it's quite clear that from the authors of [Intel SGX Explained], physical attacks cannot be "physically" prevented but can only be "economically" prevented. A more recent paper, [SoK: Hardware-supported TEEs] by _Moritz Schneider et al_, also note that in their survey of TEE designs, both in the industry and academia, none can defend against chip attacks.  

> _Invasive adversary:_ This adversary can launch invasive attacks such as de-layering the physical chip, manipulating clock signals and voltage rails to cause faults, etc., to extract secrets or force a different execution path than the intended one. For the sake of completeness, we include this adversary (`A_inv`) in our list but note that **no TEE design currently defends against such an attacker.** So, we do not discuss this attacker any further in this paper.
:::

:::warning
**Hence, chips are secure through economic incentives, not through physics.** If that is correct, using TEEs in a protocol calls for a very careful mechanism design where protocol designers take into account the cost of physically attacking the chip. For example, if we put a price tag of 1 million dollar in performing a chip attack, then a protocol using TEEs should make sure that less than 1 million dollars can be gained by performing a chip attack. Moreover, it is very important to take into account that this way of thinking does not consider attackers who wish to attack a protocol for non-economical reasons, such as breaking the privacy and/or anonymity of participants in the targeted protocol. In the case of such protocols, then it seems that current TEEs are simply not a reliable technology, as any attackers with sufficient funds and motivated to break the privacy and/or anonymity of a protocol, would be able to carry on the attack.
:::

Now, with this background in mind, it seems that it would be extremely useful to see actual chip attacks being reported by research groups, as it would help to set a price on such attacks, and the price of the attack could be use by protocol designers.

[22]: https://www.wiley.com/en-ae/Integrated+Circuit+Failure+Analysis:+A+Guide+to+Preparation+Techniques-p-9780471974017
[Intel SGX Explained]: https://eprint.iacr.org/2016/086
[SoK: Hardware-supported TEEs]: https://arxiv.org/abs/2205.12742

## Contributing to this Document
You can make edits and pull requests for https://github.com/sbellem/qtee/blob/main/qtee.md which should be a mirror of this document.
Alternatively you can also comment on or create new issues at https://github.com/sbellem/qtee/issues.
You should also be able to make comments on this document.

[^1]: Chips attacks cannot be prevented as of today (see [CHIP ATTACKS]). Making the cost of a chip attack expensive is the only current known defense mechanism. Thus, TEEs are ultimately only secure through economics.
[^2]:  Also, of relevance: https://github.com/sbellem/qtee/issues/1, https://github.com/sbellem/qtee/issues/7, https://github.com/sbellem/qtee/issues/8, [CHIP ATTACKS], and [PUFs].
[^3]: See for instance [RFC 9334](https://www.rfc-editor.org/rfc/rfc9334.html#name-security-considerations) (section 12) for security considerations when treating the topic of remote attestation.
[^4]: The reasoning is that design and implementation flaws can be fixed and can happen whether the design is open source or not, whether the supply chain is correct, etc. Hence, design and implementation bugs can be treated separately. It could be argued that an open source hardware design may benefit from a broader community and overtime will contain less bugs than a closed source design.



[Experimental relativistic zero-knowledge proofs]: https://www.nature.com/articles/s41586-021-03998-y
[Intel SGX Explained]: https://eprint.iacr.org/2016/086
[SoK: Hardware-supported TEEs]: https://arxiv.org/abs/2205.12742
[RFC 9334]: https://www.rfc-editor.org/rfc/rfc9334.html#name-security-considerations
[mechanism design]: https://en.wikipedia.org/wiki/Mechanism_design
[CHIP ATTACKS]: #Appendix-Chip-Attacks-–-What-does-it-take?
[PUFs]: #Root-of-Trust-with-PUFs
[Tiny Tapeout]: https://tinytapeout.com/
[Zero to ASIC Course]: https://zerotoasiccourse.com/
[Chips Alliance]: https://github.com/chipsalliance
[Caliptra]: https://github.com/chipsalliance/caliptra
[Google Titan]: https://opentitan.org/
[LibreSilicon]: https://libresilicon.com/
[The Silicon Salon]: https://www.siliconsalon.info/
[Logic Encryption]: https://link.springer.com/chapter/10.1007/978-3-319-49019-9_3
[HENSOLDT Cyber]: https://hensoldt-cyber.com/
[Quantum encryption with certified deletion]: https://arxiv.org/abs/1910.03551
[High-Dimensional Quantum Certified Deletion]: https://arxiv.org/abs/2304.03397
[Quantum Proofs of Deletion for Learning with Errors]: https://arxiv.org/abs/2203.01610
[Traceable Secret Sharing: Strong Security and Efficient Constructions]: https://eprint.iacr.org/2024/405
[The battle for Ring Zero]: https://pluralistic.net/2022/01/30/ring-minus-one/#drm-political-economy
[Physical One-Way Functions]: https://cba.mit.edu/docs/theses/01.03.pappuphd.powf.pdf
[ideal functionality]: https://en.wikipedia.org/wiki/Universal_composability#Ideal_functionality