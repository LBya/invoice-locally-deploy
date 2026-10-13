# InvoiceLocally - Self Hosted

Professional invoicing for musicians and freelancers.

## Installation

1.  **Purchase a License** from [Your Whop Store Link](https://whop.com/...) to get your License Key.
2.  **Download** the `docker-compose.yml` file from this repository.
3.  **Add your License Key**: In the same folder as `docker-compose.yml`, create a file named `.env` containing your key:
    ```
    LICENSE_KEY=your-actual-key-here
    ```
    Keep the `.env` file private — it holds your license key. Do not commit it to version control or share it.
4.  **Linux users only** — prepare the data folder (skip this step on Windows or macOS):
    Docker creates the folder itself on first run, but on native Linux it would then be
    owned by root and the app (which runs inside the container as user ID 1000) could not
    save your invoices. Create it up front and hand it to the app:
    ```bash
    mkdir -p ./invoicer
    sudo chown -R 1000:1000 ./invoicer
    ```
5.  **Run** the application:
    ```bash
    docker compose up -d
    ```
6.  **Open** http://localhost:8000

## Updates
The compose file tracks major version **v1** of the app (`ghcr.io/lbya/invoicer:v1`).
To update to the newest build of v1, run:
```bash
docker compose pull
docker compose up -d
```
Your data is safe: it lives in the `./invoicer` folder on your machine and is never
touched by an update.

When a **new major version** is released (v2 and beyond), this pinned file must be
updated first — re-download `docker-compose.yml` from this repository, then run the
two commands above. This is deliberate (see `docs/adr/0004-image-tag-pinning.md`):
updates arrive only after they have been pinned, so a release can never change the
app underneath you without a matching instruction here.