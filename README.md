from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from webdriver_manager.chrome import ChromeDriverManager
import time
import csv

# List of SQL injection dorks
SQL_DORKS = [
    'intext:"error in your SQL syntax"',
    'inurl:"id=" intext:"Warning: mysql_fetch_array()',
    'intext:"Warning: mysql_connect()" filetype:php',
    'inurl:"search.php?search=" intext:"Powered by phpBB"',
    'inurl:news.php?id= intext:"Powered by PHP-Fusion"'
]

def setup_driver():
    # Configure Chrome options
    options = webdriver.ChromeOptions()
    options.add_argument("--headless")  # Run in background
    options.add_argument("--disable-blink-features=AutomationControlled")
    options.add_experimental_option("excludeSwitches", ["enable-automation"])
    
    # Auto-install ChromeDriver
    service = Service(ChromeDriverManager().install())
    return webdriver.Chrome(service=service, options=options)

def google_search(driver, query, delay=5):
    try:
        driver.get("https://www.google.com")
        WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.NAME, 'q'))
        )
        
        search_box = driver.find_element(By.NAME, 'q')
        search_box.send_keys(query + Keys.RETURN)
        
        # Wait for results
        time.sleep(delay)
        
        # Get all result links
        return driver.find_elements(By.XPATH, '//div[@class="g"]//a[contains(@href, "http")]')
    except Exception as e:
        print(f"Error searching for {query}: {str(e)}")
        return []

def main():
    driver = setup_driver()
    results = []

    try:
        for dork in SQL_DORKS:
            print(f"Scanning for: {dork}")
            links = google_search(driver, dork)
            
            for link in links:
                url = link.get_attribute('href')
                if url and 'google.com' not in url:
                    results.append(url)
                    print(f"Found: {url}")

        # Save results to CSV
        with open('dork_results.csv', 'w', newline='', encoding='utf-8') as f:
            writer = csv.writer(f)
            writer.writerow(['SQL Dork Results'])
            writer.writerows([[url] for url in results])
            
    finally:
        driver.quit()
        print("Scan complete. Results saved to dork_results.csv")

if __name__ == "__main__":
    main()
