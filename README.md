# Gogs Private Fork Disclosure PoC (CVE-2026-77918)

Proof of concept for private repository metadata exposure through the fork
listing in Gogs.

Forking a public repo and flipping your copy to private is supposed to hide it.
Gogs lists every fork of a repository without checking who is allowed to see it,
so a private fork's name, owner, description, and clone URLs leak to anyone who
can see the public parent. The REST API leaks it to any account, and the web
forks page leaks it with no login at all. The API response also reports a
misleading `pull: true` for forks the caller cannot actually pull.

Tested on Gogs 0.14.3 (latest stable release).

## Requirements

- Python 3
- `requests` (`pip install requests`)
- A Gogs instance you control, with a public repo that has at least one private
  fork
- Optionally, an API token for an account that was never given access to the
  fork, to demonstrate the authenticated API leak

## Usage

```
python3 exploit.py <base_url> <owner> <repo> [--token <low-priv-token>]
```

- `<owner>` and `<repo>` identify the public parent repository
- `--token` is any account that should not see the private fork; leave it off to
  test the anonymous web path alone

Example:

```
python3 exploit.py http://127.0.0.1:3000 testadmin test-repo --token <regularuser-token>
```

## What it does

1. Calls the forks API and lists every fork, flagging the private ones and
   printing their owner, description, clone URL, and the permissions the server
   claims.
2. Fetches the web forks page with no authentication and shows that the private
   fork names appear there too.
3. Prints a verdict with the count of exposed private forks.

## Safety

Read only. It makes `GET` requests and changes nothing. Run it only against a
Gogs instance you own or have explicit permission to test.
