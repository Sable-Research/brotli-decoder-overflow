# CVE-2026-111497 — Brotli Decoder Buffer Overflow

Research notes on a buffer overflow in the Brotli decoder (`c/dec/decode.c`).

**Affected versions:** Google Brotli 1.0.3 – 1.2.0  
**Fixed in:** 1.0.10, 1.1.1, 1.2.1

This repository contains analysis only.  
No public remote code execution exploit is provided.

## Summary

A crafted Brotli-compressed stream can corrupt adjacent decoder state memory.  
Impact ranges from sensitive memory disclosure and denial of service to potential arbitrary code execution, depending on the host application and heap layout.

The vulnerability is reachable by any unauthenticated attacker who can supply Brotli-compressed data (HTTP `Content-Encoding: br`, file uploads, etc.).

## Root Cause

In affected versions the `BROTLI_SAFE` macro discards the return value of certain block-switch helpers on the fast path. When a metablock header declares a low number of block types but the stream still reaches a block-switch point, the decoder continues with stale state (`block_type_rb`, Huffman tree pointers, etc.). This results in an out-of-bounds access relative to the current Huffman tree array.

The upstream fix propagates `BROTLI_DECODER_ERROR_FORMAT_BLOCK_SWITCH` instead of silently continuing.

## Affected Versions

| Version line | Affected          | Fixed version |
|--------------|-------------------|---------------|
| 1.0.x        | 1.0.3 – 1.0.9     | 1.0.10        |
| 1.1.x        | 1.1.0             | 1.1.1         |
| 1.2.x        | 1.2.0             | 1.2.1         |

## Impact

- Memory disclosure
- Denial of service (crash / resource exhaustion)
- Potential code execution under favourable conditions

## Detection & Mitigation

- Update Brotli to 1.0.10 / 1.1.1 / 1.2.1 or newer
- Prefer applications that use a patched version of the library
- Monitor for unusual crashes in services that accept `Content-Encoding: br`
- Restrict or disable Brotli decompression for untrusted input where possible


## Disclaimer

This repository is published by **Sable Research** for educational and defensive research purposes only.  

The information contained herein is based on publicly available sources and is intended to support understanding of the vulnerability and its impact.  

Sable Research does not condone or encourage any unauthorized or illegal use of this information.  
Do not test or exploit systems that you do not own or have explicit written permission to assess.
