#### Managing pairing

The `sc_auth` utility (_smart card authorization setup script_) bundled by macOS can be used for a variety of SmartCard related operations.

1. List all SmartCard hashes for user `foobar`:

  ```sh
  ❯ sc_auth list -u foobar
  ```

2. Unpair a SmartCard hash `bar` from user `foobar`:

  ```sh
  ❯ sc_auth unpair -u foobar -h bar
  ```

3. Unpair all SmartCards from user `foobar`:

  ```sh
  ❯ sc_auth unpair -u foobar
  ```

4. Disable the SmartCard Pairing UI whenever a new SmartCard is entered:

  ```sh
  ❯ sc_auth pairing_ui -s disable
  ```
