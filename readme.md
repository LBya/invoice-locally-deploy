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
4.  **Run** the application:
    ```bash
    docker-compose up -d
    ```
5.  **Open** http://localhost:8000

## Updates
To update to the latest version, run:
```bash
docker-compose pull
docker-compose up -d
```