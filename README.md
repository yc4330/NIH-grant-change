# Data Analysis for the Project

## Scopus API for Chinese-descent Scientist Departure from the US

The Scopus API, offered by Elsevier, allows developers to access the Scopus database, which contains abstracts and citations for academic journal articles. The API key is available through Columbia University and the Scopus Author API can only be accessed from within the university's network.

### Data Extraction

The primary objective is to identify scientists of Chinese descent who depart from the US each year. "Leaving the US" refers to an author who previously had a US affiliation but now has a non-US affiliation. The year of departure is determined by the publication date of the first paper after they ceased using their US affiliation.

Due to the large volume of papers, I firstly narrow the focus to names of Chinese descent to reduce the scope. Next, I further filter by selecting authors whose current affiliation is non-US and who previously had a US affiliation. Finally, I review the paper information to obtain the dates of each affiliation, identifying the year when the author left the US.

```
# Parameters for the initial API request
params = {
    'query': query,
    'apiKey': api_key,
    'httpAccept': 'application/json'
}
```

#### 1. Compile a list of Chinese surnames.
Chinese surnames can be collected from [Wikipedia](https://en.wikipedia.org/wiki/List_of_common_Chinese_surnames), using Hanyu Pinyin (the system of Chinese romanization mostly used by mainland Chinese scientists) and Wade-Giles (the system mostly used by Cantonese-speaking and Taiwanese scientists). This methodology results in the non-counting of Chinese-descent scientists who have changed their surnames (usually females after marriage), leading to an undercount. 

Note: Scopus supports names that contain non-English characters, such as "ü", which are correctly displayed in the JSON returned by the API. Therefore, the surname list should also contain non-English characters and single quotation mark.

Methodology Citation: Yu Xie, Xihong Lin, Ju Li, Qian He, Junming Huang, Caught in the Crossfire: Fears of Chinese-American Scientists, Proceedings of the National Academy of Sciences, 120 (27) e2216248120 (2023).

#### 2. Create a list of Scopus Author IDs corresponding to each Chinese surname.
Having a unique author identifier is crucial for tracking each author's activity, and Scopus provides this functionality through its author ID.

Search by author surname and get every scopus id. Create a whole list of scopus id of scientists that have a Chinese surname.

```
# Base URL for the Scopus Author Search API
author_search_base_url = 'https://api.elsevier.com/content/search/author'

# Query to search for the author surname
query = 'AUTHLASTNAME(Surname)'

# Make the initial API request to search for the author
response = requests.get(author_search_base_url, params=params)

# Function to extract author Scopus IDs from the search results
def extract_author_scopus_ids(results):
    entries = results.get('search-results', {}).get('entry', [])
    print(len(entries))
    author_ids = []
    for entry in entries:
        author_ids.append(entry.get('dc:identifier').split(':')[1])
    return author_ids

# Extract author Scopus IDs from the initial search results
author_scopus_ids = extract_author_scopus_ids(results)
```

#### 3. Filter the authors who have left the US based on their affiliation history.
Search author using the author_id. Determine if the country is not the US from the "affiliation-current" information. Then, examine the "affiliation history" URLs to check if any include the US as the country. If both conditions are satisfied, save the Scopus ID, the historical affiliation ID and country to the targeted author list.


#### 4. Search for the year information using Scopus Author IDs.
Search paper using the author_id. For each paper, search using the scopus id for paper to get date and fields of authors and affiliations. In the list of authors, find the AUID equal to the author_id to get the year for that particular affiliation.

<details>
  <summary>Click to unfold the code</summary>

```
# The Scopus ID of the author
scopus_id = '35228272300'

# Base URL for the Scopus Author Retrieval API
author_retrieval_base_url = f'https://api.elsevier.com/content/author/author_id/{scopus_id}'

# Query to search for papers by the specific author ID
query = f'AU-ID({author_id})'
params = {'query': query}
response = requests.get(search_base_url, params=params)

def get_all_papers(query, api_key):
    all_papers = []
    start = 0
    count = 200

    while True:
        params['start'] = start
        response = requests.get(search_base_url, headers=headers, params=params)
        if response.status_code == 200:
            results = response.json()
            papers = results.get('search-results', {}).get('entry', [])
            all_papers.extend(papers)

            if len(papers) < count:
                break

            start += count
        else:
            print(f"Error: {response.status_code}, {response.text}")
            break

    return all_papers

papers = get_all_papers(query, api_key)
```
</details>

#### 5. Organize the cleaned data into a table.
The table should contain such columns:

| Scopus Author ID | Current Country | Departure Year | Number of papers before departure| Subject Areas |
| ------- | ------- | ------- | ------- | ------- |

Number of papers before departure can be categorized as Junior or Senior for further analysis.Subject Areas contain all the subjects listed by Scopus API. It can be categorized into several disciplines using GPT. 

A table of 25202 rows is already obtained from the same paper for data before 2022, but using different author ID identifier. Each row is a Chinese-descent scientist left the US.

| Microsoft Academic Graph Author ID | destination country or region | move year | discipline|
| ------- | ------- | ------- | ------- |
| 209030549 | Taiwan | 1996 | Life science |
| 351961669 | Singapore | 2014 | Social sciences |


### Data Analysis

### Visualization

## NIH RePORTER Project Data for PI Nationality Perdicted by Name

The NIH [RePORTER]("https://reporter.nih.gov/exporter/projects") (Research Portfolio Online Reporting Tools) Project Data is a database that provides detailed information about research projects funded by the National Institutes of Health (NIH). It includes data on project descriptions, funding amounts, principal investigators, institutions, study sections, and related publications and patents. 


### Data Extraction

NIH grants are primarily US-focused. For instance, in the 2023 data, only 2 projects were awarded to Chinese institutions, compared to 80,936 projects awarded to US organizations out of a total of 84,793 projects. Therefore, my main objective is to focus on US-based scientists of Chinese descent and assess whether they have relatively fewer opportunities to become NIH principal investigators (PIs) compared to their white or other Asian colleagues.

While NIH provides open data on [the races of PIs]("https://report.nih.gov/nihdatabook/category/25"), it does not include counts by descent or nationality. Since this research is focused on China, I will need to compare the PI names with the Chinese surname lists obtained earlier.

Additional information on nationality or country of origin, beyond China, can be obtained using the Python library [name2nat]("https://github.com/Kyubyong/name2nat"). (However, my jupyter notebook cell always collapse when importing this library or any NLP libraries, which I will need to ask for help from data professor.)

And then I can have the funding information for each project.

### Data Analysis

| Year | Non-Asian | Asian | Chinese|
| ------- | ------- | ------- | ------- |
|2023|52131|32662|9499|
|2022|52050|31680|9078|

### Visualization

## US-China Collaboration from NIH Paper 

my.query <- '("China"[Affiliation] AND "united states"[Affiliation]) AND (NIH[Grants and Funding])' 

### Visualization

See [here]("https://yc4330.github.io/Master-Project"). Slide to explore how varying ranges of scholars impact the shape of the line graph. Scholars are sorted in descending order based on the total number of collaborative papers.

<iframe src="https://yc4330.github.io/Master-Project" width="600" height="400"></iframe>

### Analysis

Is it because of covid? The answer is no when compared with the data with other countries.

<img src="image/1721063009069.jpg" width="400">

<img src="image/1721063053517.jpg" width="400">

### 