# Password Management

## How It Works

HCI stores the dashboard password in `.env` as `HERMES_CONTROL_PASSWORD`.

The password can be stored in two formats:

### Bcrypt Hashed (Recommended)
```
HERMES_CONTROL_PASSWORD=$2b$10$xJ8kL3mN9pQ2rS5tU7vW1y...
```
- Starts with `$2b$` or `$2a$`
- Compared using `bcrypt.compareSync()` — timing-safe, irreversible
- If `.env` leaks, attacker cannot recover the password

### Plaintext (Legacy)
```
HERMES_CONTROL_PASSWORD=mysecretpassword123
```
- Compared using `crypto.timingSafeEqual()` — timing-safe but reversible
- If `.env` leaks, attacker can read the password directly

The server auto-detects which format is used on login.

---

## First-Time Setup

When you run `bash install.sh`:
1. A random 24-character password is generated
2. It's hashed with bcrypt (10 rounds)
3. The hash is saved to `.env`
4. The plaintext password is shown ONCE — **save it somewhere safe**

If bcrypt is not available (e.g., `npm install` hasn't finished), the password is saved as plaintext. Run `bash reset-password.sh` later to hash it.

---

## Reset Password

### Option 1 — Interactive (asks for password)
```bash
cd hermes-control-interface
bash reset-password.sh
```

### Option 2 — Direct (pass password as argument)
```bash
bash reset-password.sh "my-new-password"
```

### Option 3 — Via npm
```bash
npm run reset-password
```

### What happens:
1. You enter a new password
2. It's hashed with bcrypt (10 rounds)
3. The hash replaces `HERMES_CONTROL_PASSWORD` in `.env`
4. Restart the server for changes to take effect

### After reset:
```bash
# If running directly
npm start

# If using systemd
sudo systemctl restart hermes-control
```

---

## Check Current Password Format

```bash
grep HERMES_CONTROL_PASSWORD .env
```

- Starts with `$2b$` or `$2a$` → bcrypt hashed (secure)
- Anything else → plaintext (run `reset-password.sh` to hash)

---

## Security Notes

- The bcrypt hash is **one-way** — you cannot recover the plaintext from it
- If you forget your password, you MUST reset it (there's no recovery)
- Keep your `.env` file permissions at `600` (`chmod 600 .env`)
- Never commit `.env` to git (it's in `.gitignore` by default)
- `HERMES_CONTROL_SECRET` is separate — it's used for auth token signing, not password comparison
