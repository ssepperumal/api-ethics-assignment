API Ethics Assignment

Task 1 – PII Classification

ok so i went through each field and here's what i think:

- full_name – this is direct PII, like its literally the persons name. i would just drop it, we dont need it for research
- email – also direct PII, can directly contact the person with this. drop it
- date_of_birth – direct PII. instead of keeping the full date i'd convert it to just age or like an age range (20-30, 30-40 etc). that way its still useful
- zip_code – this one is indirect PII, alone its not enough to identify someone but combined with other fields it can be. i'd mask it by only keeping first 3 digits
- job_title – indirect PII. could help identify someone if its a rare job. i'd either drop it or generalize it like just say "healthcare worker" instead of the exact title
- diagnosis_notes – indirect PII but risky because free text can have names or dates hidden in it. i'd redact anything that looks personal and replace patient names with a random ID



Task 2 – Problems in the Script

Problem 1 – scraping 100 pages with a free key

the script just loops through 100 pages without checking if thats even allowed. free tier APIs usually have limits and doing this probably breaks the terms of service. also theres no delay between requests which can get the key banned.

fixed version:

```python
import requests
import time

API_URL = "https://healthstats-api.example.com/records"
API_KEY = "free_tier_key_abc123"

records = []
for page in range(1, 11):  # only doing 10 pages, free tier limit
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})
    
    if response.status_code == 429:  # means we hit the rate limit
        print("rate limit hit, stopping")
        break
    
    data = response.json()
    records.extend(data["results"])
    time.sleep(1)  # wait 1 sec between requests
```



Problem 2 – storing everything permanently with raw PII

the comment literally says "store all records permanently" which is a big problem. you're not supposed to keep personal health data forever, and you definitely shouldnt store names and emails before sharing with a research partner.

fixed version:

```python
from datetime import datetime, timedelta

def anonymise(record):
    # remove the obvious personal stuff
    record.pop("full_name", None)
    record.pop("email", None)
    
    # turn date of birth into age group instead
    if "date_of_birth" in record:
        dob = datetime.strptime(record["date_of_birth"], "%Y-%m-%d")
        age = (datetime.utcnow() - dob).days // 365
        record["age_group"] = f"{(age//10)*10}-{(age//10)*10+9}"
        del record["date_of_birth"]
    
    # only keep first 3 digits of zip
    if "zip_code" in record:
        record["zip_code"] = record["zip_code"][:3] + "**"
    
    return record

clean_records = [anonymise(r) for r in records]

# store with a 1 year limit, not permanently
save_to_database(clean_records, retention_days=365)
```
