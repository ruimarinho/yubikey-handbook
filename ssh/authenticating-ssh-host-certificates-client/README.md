## Authenticating SSH Host Certificates (client)

Similar to the [SSH User Certificates story](#remote-ssh-authentication-via-user certificates), it is also possible to authenticate hosts via client side certificate authority authentication. An SSH Server Certificate Authority signs server certificates and the client only needs to be aware of the CA's public key.

1. Using an air gapped computer, generate the server certificate authority:

  ```sh
  ❯ ssh-keygen -C "SSH Server Certificate Authority" -f sshserver.root.ca
  ```

2. Sign the host public key using the server certificate authority:

  ```sh
  ❯ ssh-keygen -s sshserver.root.ca -I <identity> -h -n <hostname> -V +52w /etc/ssh/ssh_host_rsa_key.pub

  Signed host key /etc/ssh/ssh_host_rsa_key-cert.pub: id "foobar.com-key" serial 0 for foobar.com valid from 2016-12-10T00:10:00 to 2017-12-09T00:10:10
  ```

3. Update `/etc/ssh/sshd_config` to include the new host certificate:

  ```sh
  HostCertificate /etc/ssh/ssh_host_rsa_key-cert.pub
  ```

4. Add the certificate authority to the local ssh client (`~/.ssh/known_hosts`):

  ```
  @cert-authority *.foobar.com <content of sshserver.root.ca.pub>
  ```
