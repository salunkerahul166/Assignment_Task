# Assignment_Task
Assingement Task
import requests
from bs4 import BeautifulSoup
import time

BASE_URL = "https://rera.odisha.gov.in"
PROJECT_LIST_URL = f"{BASE_URL}/projects/project-list"

headers = {
    "User-Agent": "Mozilla/5.0"
}

def get_first_6_project_links():
    response = requests.get(PROJECT_LIST_URL, headers=headers)
    soup = BeautifulSoup(response.content, "html.parser")

    projects = soup.find_all("a", string="View Details", limit=6)
    project_links = [BASE_URL + project["href"] for project in projects]

    return project_links

def scrape_project_details(url):
    response = requests.get(url, headers=headers)
    soup = BeautifulSoup(response.content, "html.parser")

    def get_text_by_label(label):
        el = soup.find("th", string=label)
        if el and el.find_next_sibling("td"):
            return el.find_next_sibling("td").get_text(strip=True)
        return "N/A"

    details = {
        "Rera Regd. No": get_text_by_label("RERA Regd. No."),
        "Project Name": get_text_by_label("Project Name"),
        "Promoter Name": get_text_by_label("Company Name"),
        "Promoter Address": get_text_by_label("Registered Office Address"),
        "GST No": get_text_by_label("GST No."),
        "Detail Page": url
    }
    return details

def main():
    project_links = get_first_6_project_links()
    all_details = []

    for link in project_links:
        print(f"Scraping: {link}")
        details = scrape_project_details(link)
        all_details.append(details)
        time.sleep(2)  # Be polite to the server

    for i, project in enumerate(all_details, start=1):
        print(f"\n--- Project {i} ---")
        for key, value in project.items():
            print(f"{key}: {value}")

if __name__ == "__main__":
    main()
