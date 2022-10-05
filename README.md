# Vulnerable NodeJS Application

Originaly this project was created by [appsecengineer](https://github.com/appsecengineer/gh-actions-nodejsscan) with the goal of demonstrating NodeJS scan via Github Actions. I added additional jobs via Jenkins pipeline.

The following Github Actions were added:
* Snyk Container scanning
* Snyk Code scanning (SAST)
* Snyk Dependency scanning (SCA)
* Semgrep scanning (CI)
* Anchore Grype scanning

The following Jenkins jobs were added:
* Create SBOM with Syft
* Upload the SBOM to Dependenc-track to analyse vulnerabilities (synchronous)
* Start Grype scan using the SBOM
