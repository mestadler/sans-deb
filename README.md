Packaging with fpm and Scanning with TrivyThis guide provides a step-by-step workflow on how to package binaries into .deb and .rpm formats using fpm, and how to scan the artifacts for vulnerabilities using Trivy.Table of ContentsInstall PrerequisitesPrepare Your BinariesScan Binaries Before PackagingCreate the .deb PackageCreate the .rpm PackageFinal Verification ScanLicense1. Install PrerequisitesBefore you begin, you need to install fpm and Trivy.a. Install fpmfpm is a Ruby gem, so you will need to have Ruby and its package manager, gem, installed on your system.On Debian/Ubuntu:sudo apt-get update
sudo apt-get install ruby ruby-dev build-essential
On RHEL/CentOS/Fedora:sudo yum install ruby ruby-devel rpm-build
Once Ruby is installed, you can install fpm:sudo gem install fpm
b. Install TrivyThe recommended way to install Trivy is via their official script. For other methods, refer to the official Trivy documentation.curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
2. Prepare Your BinariesGather all the compiled binaries you wish to package into a single, dedicated directory. For this guide, we will assume your binaries are located in a directory named my-app-binaries/.my-app-binaries/
├── binary-one
├── binary-two
└── binary-three
3. (Recommended) Scan Binaries Before PackagingBefore packaging, it's crucial to scan your raw binary files for any known vulnerabilities. Use the trivy fs (filesystem) command to scan the entire directory containing your binaries.# Navigate to the parent directory of your binaries folder
cd /path/to/

# Scan the directory
trivy fs ./my-app-binaries/
Review the output from Trivy and address any critical vulnerabilities before proceeding.4. Create the .deb PackageOnce you are confident the binaries are secure, create the .deb package.# Navigate to the directory containing your binaries
cd /path/to/my-app-binaries/

# Run the fpm command to create the .deb package
fpm -s dir -t deb \
    -n <package-name> \
    -v <version> \
    --iteration <build-number> \
    --architecture <architecture> \
    --description "A description for your package." \
    --prefix /usr/local/bin \
    .
5. Create the .rpm PackageThe command to create an .rpm package is nearly identical.# Navigate to the directory containing your binaries
cd /path/to/my-app-binaries/

# Run the fpm command to create the .rpm package
fpm -s dir -t rpm \
    -n <package-name> \
    -v <version> \
    --iteration <build-number> \
    --architecture <architecture> \
    --description "A description for your package." \
    --prefix /usr/local/bin \
    .
6. (Optional) Final Verification Scan of PackagesAs a final check, scan the generated package files. This ensures the packaging process did not introduce any new issues.Scan the .deb package:trivy fs <package-name>_<version>-<build-number>_<architecture>.deb
Scan the .rpm package:trivy fs <package-name>-<version>-<build-number>.<architecture>.rpm
After running these commands, you can confidently distribute your scanned and verified packages.7. LicenseThis project is licensed under the MIT License - see the LICENSE file for details.
