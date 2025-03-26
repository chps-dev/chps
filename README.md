# Container Hardening Priorities (CHPs)

<img align="right" width="200" src="https://github.com/chps-dev/chps/blob/main/img/chps-logo.png" alt="CHPs
logo - a bowl of tortilla chips">

This repository provides guidance on objectively classifying container images for security hardening. The [CHPs specification](./CHPs.md) details security criteria across 4 "vectors":

- **Minimalism** – Reducing software packages and complexity  
- **Provenance** – Verifying source and authenticity  
- **Configuration & Metadata** – Applying secure defaults and best practices  
- **Vulnerabilities** – Tracking known vulnerabilities at any point in time  

## Why CHPs?

CHPs help DevOps teams, image authors, and security practitioners quickly see where container images meet or miss security hardening measures. Users can self-assess images and display badges that reflect their security posture.

## Getting Started

1. **Read the Spec**: See [CHPs.md](./CHPs.md) for the complete classification levels.
2. **Assess Your Image**: See how well your images rank across the criteria. Use the [CHPs Scorer](https://github.com/chps-dev/chps-scorer) project to do this automatically. 
3. **Badge Your Projects**: Add badges showing how your images score across minimalism, provenance, configuration, and vulnerabilities.

## Contributing

Please open issues or submit pull requests to help refine the specification or suggest new criteria.

## License

All CHPs specification content is provided under the
[Community Specification License 1.0](License.md).

