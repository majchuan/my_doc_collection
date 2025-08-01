# 📧 Setting Up Zoho Mail with Your Custom Domain

**Date:** 2025-08-01

---

## ✅ Step-by-Step Guide to Set Up Zoho Mail

### 1. Sign Up for Zoho Mail

- Go to [https://zoho.com/mail/zohomail-pricing.html](https://zoho.com/mail/zohomail-pricing.html)
- Choose:
  - **Mail Lite** ($1/user/month) – Recommended
  - OR **Forever Free Plan** – Up to 5 users, 5GB per user
- Click **Sign Up Now**
- Choose **Sign up with a domain I already own**
- Enter your domain (e.g., `yourdomain.com`)

---

### 2. Verify Your Domain

Zoho will ask you to verify domain ownership.

- Add a **TXT record** in your DNS settings:
  - Type: TXT
  - Name/Host: `@`
  - Value: `zoho-verification=xxxxxxxxx` (provided by Zoho)
- If using **DigitalOcean DNS**:
  - Go to **Networking > Domains > YourDomain**
  - Add the TXT record
- Click **Verify** in Zoho after saving the record (may take a few minutes)

---

### 3. Create Email Accounts

- After domain verification, create users like:
  - `yourname@yourdomain.com`
- Set passwords, aliases, or groups as needed

---

### 4. Update MX Records in Your DNS

Delete Gmail Workspace records and add these:

| Type | Name | Priority | Value            |
|------|------|----------|------------------|
| MX   | @    | 10       | mx.zoho.com      |
| MX   | @    | 20       | mx2.zoho.com     |
| MX   | @    | 50       | mx3.zoho.com     |

In **DigitalOcean**:
- Go to **Networking > Your Domain**
- Remove old MX records and add the Zoho records above

⏱ MX record changes usually take 1–24 hours to propagate.

---

### 5. Add SPF, DKIM, and DMARC (Important for Deliverability)

**SPF (TXT):**
```
v=spf1 include:zoho.com ~all
```

**DKIM:** Generate from Zoho Admin Panel and add as TXT.

**DMARC (optional, TXT):**
```
v=DMARC1; p=none; rua=mailto:dmarc@yourdomain.com
```

---

### 6. Test Your Zoho Email

- Log into [https://mail.zoho.com](https://mail.zoho.com)
- Send and receive test emails to/from Gmail or Outlook

---

### 7. Migrate Old Gmail Emails (Optional)

Use Zoho’s built-in migration tool:

- Admin Console → **Data Migration**
- Choose Gmail/Google Workspace → authenticate
- Select mail folders to migrate

---

### 🔄 When to Disable Gmail Workspace

- Wait **48–72 hours** after updating MX records before canceling Gmail Workspace.
- This ensures mail delivery is fully switched and no emails are lost.

---

### 🧾 Summary Table

| Step            | Action                                    |
|-----------------|--------------------------------------------|
| Sign up         | Register on Zoho and verify domain         |
| DNS changes     | Add TXT + replace MX records               |
| Create mailbox  | Set up user accounts like name@domain.com  |
| Email security  | Set up SPF, DKIM, DMARC                    |
| Testing         | Check deliverability with test emails      |
| Migrate         | Optionally move emails from Gmail          |
| Cancel Gmail    | After 2–3 days of successful testing       |

---

**Done!** 🎉 You’re now using Zoho Mail with your custom domain!
