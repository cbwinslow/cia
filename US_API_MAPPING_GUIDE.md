# 🔌 US API Integration Mapping Guide

**Version:** 1.0  
**Date:** 2025-12-28  
**Purpose:** Technical reference for Swedish to US API migration

---

## 📋 Overview

This document provides detailed mapping between Swedish data APIs (Riksdagen, Val, ESV) and their US Congressional equivalents (Congress.gov, ProPublica, FEC, USAspending). It serves as a technical reference for developers implementing the US adaptation.

---

## 🏛️ Parliamentary/Congressional Data

### Riksdagen API → Congress.gov + ProPublica APIs

#### Member/Person Data

**Swedish Source: Riksdagen Person API**
```http
GET http://data.riksdagen.se/personlista/
```

**US Equivalent: Congress.gov Members API**
```http
GET https://api.congress.gov/v3/member
GET https://api.congress.gov/v3/member/{bioguideId}

Headers:
  X-API-Key: {your-api-key}

Response Fields:
- bioguideId (unique identifier)
- firstName, lastName, middleName
- party (D, R, I, etc.)
- state (2-letter code)
- district (number, null for Senators)
- currentMember (boolean)
- terms (array of service periods)
```

**US Alternative: ProPublica Congress API**
```http
GET https://api.propublica.org/congress/v1/{congress}/senate/members.json
GET https://api.propublica.org/congress/v1/{congress}/house/members.json
GET https://api.propublica.org/congress/v1/members/{member-id}.json

Headers:
  X-API-Key: {your-propublica-api-key}

Response Fields:
- id (ProPublica member ID)
- first_name, last_name, middle_name
- party (D, R, I)
- state
- district (House only)
- seniority (years in office)
- next_election
- leadership_role
- twitter_account, facebook_account
- votes_with_party_pct
- missed_votes_pct
```

**Field Mapping:**

| Swedish Field | Congress.gov Field | ProPublica Field | Notes |
|--------------|-------------------|------------------|-------|
| `intresseId` | `bioguideId` | `id` | Unique member identifier |
| `fornamn` | `firstName` | `first_name` | First name |
| `efternamn` | `lastName` | `last_name` | Last name |
| `parti` | `party` | `party` | Party abbreviation |
| `valkrets` | `state` + `district` | `state` + `district` | Electoral district |
| `status` | `currentMember` | N/A | Current service status |
| N/A | N/A | `votes_with_party_pct` | Party loyalty metric |
| N/A | N/A | `missed_votes_pct` | Attendance metric |

#### Voting Data

**Swedish Source: Riksdagen Voting API**
```http
GET http://data.riksdagen.se/voteringlista/
GET http://data.riksdagen.se/votering/{voteid}
```

**US Equivalent: Congress.gov Votes API**
```http
GET https://api.congress.gov/v3/vote/{congress}/{chamber}/{session}/{vote-number}
GET https://api.congress.gov/v3/vote/{congress}/{chamber}

Parameters:
- congress: 118, 117, etc.
- chamber: house, senate
- session: 1, 2
- vote-number: roll call number

Response Fields:
- congress
- session  
- chamber
- rollCallNumber
- question (motion text)
- result (Passed, Failed, etc.)
- members (array of vote positions)
  - bioguideId
  - name
  - party
  - state
  - votePosition (Yea, Nay, Present, Not Voting)
```

**US Alternative: ProPublica Votes API**
```http
GET https://api.propublica.org/congress/v1/{congress}/{chamber}/sessions/{session}/votes/{roll-call}.json
GET https://api.propublica.org/congress/v1/{congress}/{chamber}/votes/recent.json

Response Fields:
- roll_call
- chamber
- session
- congress
- vote_date
- question
- result
- total (yes, no, not_voting counts)
- positions (array)
  - member_id
  - name
  - party
  - state
  - vote_position
```

**Field Mapping:**

| Swedish Field | Congress.gov Field | ProPublica Field | Notes |
|--------------|-------------------|------------------|-------|
| `voteringId` | `rollCallNumber` | `roll_call` | Vote identifier |
| `rm` (Riksmöte) | `session` | `session` | Legislative session |
| `punkt` | `rollCallNumber` | `roll_call` | Agenda item number |
| `votering` | `question` | `question` | Vote description |
| `utfall` | `result` | `result` | Vote outcome |
| `Ja` count | `yeas` | `total.yes` | Yes votes |
| `Nej` count | `nays` | `total.no` | No votes |
| `Avstår` count | `present` | `total.present` | Present but not voting |
| `Frånvarande` | `notVoting` | `total.not_voting` | Absent |

#### Document/Bill Data

**Swedish Source: Riksdagen Document API**
```http
GET http://data.riksdagen.se/dokumentlista/
GET http://data.riksdagen.se/dokument/{dokid}
```

**US Equivalent: Congress.gov Bills API**
```http
GET https://api.congress.gov/v3/bill/{congress}/{billType}
GET https://api.congress.gov/v3/bill/{congress}/{billType}/{billNumber}

Parameters:
- congress: 118, 117, etc.
- billType: hr, s, hjres, sjres, hconres, sconres, hres, sres
- billNumber: bill number

Response Fields:
- congress
- type (bill type)
- number
- title (official title)
- introducedDate
- latestAction
- sponsors (array)
  - bioguideId
  - firstName, lastName
  - party, state
- cosponsors (array)
- committees (array)
- subjects (policy areas)
- textVersions (array of bill text URLs)
```

**Field Mapping:**

| Swedish Field | Congress.gov Field | Notes |
|--------------|-------------------|-------|
| `dokid` | `{congress}-{billType}-{billNumber}` | Composite identifier |
| `typ` | `type` | Document/bill type |
| `rubrik` | `title` | Title |
| `datum` | `introducedDate` | Introduction date |
| `dokument` | `textVersions[].url` | Full text URL |
| `undertecknare` | `sponsors` | Primary sponsor |
| N/A | `cosponsors` | Cosponsors (US specific) |
| `utskott` | `committees` | Committee assignments |

#### Committee Data

**Swedish Source: Riksdagen Committee API**
```http
GET http://data.riksdagen.se/utskottsforslag/
```

**US Equivalent: Congress.gov Committee API**
```http
GET https://api.congress.gov/v3/committee/{congress}/{chamber}
GET https://api.congress.gov/v3/committee/{chamber}/{committeeCode}

Response Fields:
- systemCode (unique identifier)
- name
- chamber
- committeeType
- parent (for subcommittees)
- subcommittees (array)
- history (array of activities)
  - date
  - text
```

**Field Mapping:**

| Swedish Field | Congress.gov Field | Notes |
|--------------|-------------------|-------|
| `utskottId` | `systemCode` | Committee identifier |
| `utskottsnamn` | `name` | Committee name |
| `organ` | `chamber` | House or Senate |
| N/A | `subcommittees` | Subcommittees (US specific) |

---

## 🗳️ Election Data

### Val (Swedish Election Authority) → FEC (Federal Election Commission)

#### Party/Political Organization Data

**Swedish Source: Val Party API**
```http
GET http://data.val.se/partier/
```

**US Equivalent: FEC Committees API**
```http
GET https://api.open.fec.gov/v1/committees/
GET https://api.open.fec.gov/v1/committee/{committee_id}/

Parameters:
- organization_type: P (Presidential), S (Senate), H (House), N (PAC)
- designation: P (Principal), A (Authorized)
- party: DEM, REP, IND, etc.

Response Fields:
- committee_id
- name
- party
- committee_type
- designation
- state
- treasurer_name
- organization_type
```

**Field Mapping:**

| Swedish Field | FEC Field | Notes |
|--------------|-----------|-------|
| `partiId` | `committee_id` | Party/committee identifier |
| `partinamn` | `name` | Party name |
| `partibeteckning` | `party` | Party abbreviation |
| N/A | `committee_type` | US specific (principal, authorized, etc.) |

#### Election Results

**Swedish Source: Val Election API**
```http
GET http://data.val.se/val/
```

**US Equivalent: FEC Election Results API**
```http
GET https://api.open.fec.gov/v1/elections/
GET https://api.open.fec.gov/v1/elections/search/

Parameters:
- cycle: 2024, 2022, 2020, etc.
- office: H (House), S (Senate), P (President)
- state: 2-letter state code
- district: district number (House only)

Response Fields:
- cycle
- office
- state
- district
- candidate_id
- candidate_name
- party
- votes
- incumbent_challenge (I, C, O)
```

**US Alternative: ElectionResults via House/Senate Clerk**
- House: https://clerkpreview.house.gov/
- Senate: https://www.senate.gov/legislative/LIS_MEMBER_LISTING.htm

**Field Mapping:**

| Swedish Field | FEC Field | Notes |
|--------------|-----------|-------|
| `valAr` | `cycle` | Election year |
| `valkrets` | `state` + `district` | Electoral district |
| `parti` | `party` | Party |
| `kandidat` | `candidate_name` | Candidate |
| `roster` | `votes` | Vote count |

#### Campaign Finance

**Swedish Source: Not Available**

**US Equivalent: FEC Campaign Finance API**
```http
GET https://api.open.fec.gov/v1/candidate/{candidate_id}/totals/
GET https://api.open.fec.gov/v1/committee/{committee_id}/filings/
GET https://api.open.fec.gov/v1/schedules/schedule_a/

Response Fields:
- candidate_id
- committee_id
- total_receipts
- total_disbursements
- cash_on_hand
- debts_owed
- contributions_by_individual
- contributions_by_pac
- contributions_by_party_committee
```

**New Capability (No Swedish Equivalent):**

This is a significant enhancement for US adaptation:
- Individual contribution tracking
- PAC contribution analysis  
- Independent expenditure monitoring
- Coordination detection
- Debt and loan tracking

---

## 💰 Government Financial Data

### ESV (Swedish Financial Management Authority) → USAspending.gov

#### Government Agency Financial Data

**Swedish Source: ESV Government Body API**
```http
GET http://www.esv.se/ (scraped data)
```

**US Equivalent: USAspending.gov Agency API**
```http
GET https://api.usaspending.gov/api/v2/agency/{toptier_agency_code}/
GET https://api.usaspending.gov/api/v2/federal_accounts/

Response Fields:
- toptier_agency_code
- agency_name
- current_total_budget_authority
- obligations_by_award_type
- obligations_by_object_class
- agency_data_by_year
```

**Field Mapping:**

| Swedish Field | USAspending Field | Notes |
|--------------|-------------------|-------|
| `organisationsnummer` | `toptier_agency_code` | Agency identifier |
| `namn` | `agency_name` | Agency name |
| `budget` | `current_total_budget_authority` | Budget authority |
| `utfall` | `obligations_by_award_type` | Actual spending |
| N/A | `obligations_by_object_class` | Spending by category (US specific) |

#### Federal Spending by Category

**Swedish Source: Not Available**

**US Equivalent: USAspending Awards API**
```http
GET https://api.usaspending.gov/api/v2/search/spending_by_award/
GET https://api.usaspending.gov/api/v2/spending/

Parameters:
- award_type: contracts, grants, direct_payments, loans
- recipient_name
- naics_code (industry classification)
- time_period (start/end dates)

Response Fields:
- award_type
- recipient_name
- award_amount
- awarding_agency
- period_of_performance
- place_of_performance
```

**New Capabilities (No Swedish Equivalent):**
- Contract award tracking
- Grant distribution analysis
- Loan program monitoring
- Direct payment tracking
- Geographic spending analysis

---

## 🌍 Economic Indicators (Unchanged)

### World Bank API

Both Swedish and US implementations use the same World Bank API:

```http
GET https://api.worldbank.org/v2/country/{country}/indicator/{indicator}

Country Codes:
- Sweden: SWE
- United States: USA

Common Indicators:
- NY.GDP.MKTP.CD (GDP)
- SP.POP.TOTL (Population)
- SL.UEM.TOTL.ZS (Unemployment)
- FP.CPI.TOTL (Inflation)
```

**Configuration Change Only:**
```properties
# Change default country
world.bank.default.country=USA  # was: SWE
```

---

## 🔑 API Authentication Requirements

### API Keys Required

| API | Registration URL | Tier | Rate Limits |
|-----|-----------------|------|-------------|
| **Congress.gov** | https://api.congress.gov/sign-up/ | Free | Reasonable limits |
| **ProPublica Congress** | https://www.propublica.org/datastore/api/propublica-congress-api | Free | 5,000 requests/day |
| **FEC** | https://api.open.fec.gov/developers/ | Free | 1,000 requests/hour |
| **USAspending** | N/A | Free | No strict limits |
| **World Bank** | N/A | Free | No authentication |

### API Key Configuration

```properties
# application.properties
congress.api.key=${CONGRESS_API_KEY}
propublica.api.key=${PROPUBLICA_API_KEY}
fec.api.key=${FEC_API_KEY}

# Environment variables
export CONGRESS_API_KEY=your_congress_api_key
export PROPUBLICA_API_KEY=your_propublica_api_key
export FEC_API_KEY=your_fec_api_key
```

---

## 📊 Data Volume Estimates

### Congressional Data

| Data Type | Records/Year | Storage | Update Frequency |
|-----------|-------------|---------|------------------|
| Members | ~535 | < 1 MB | Per Congress (2 years) |
| Bills | ~10,000 | ~500 MB | Daily |
| Votes | ~1,000 | ~100 MB | Daily |
| Committee Assignments | ~2,000 | < 10 MB | Per Congress |

### Election Data

| Data Type | Records/Cycle | Storage | Update Frequency |
|-----------|--------------|---------|------------------|
| Candidates | ~2,000 | ~10 MB | Per election cycle |
| Election Results | ~1,500 | ~20 MB | After elections |
| Campaign Finance | ~100,000 | ~1 GB | Quarterly |
| Independent Expenditures | ~50,000 | ~500 MB | Real-time during election season |

### Federal Spending Data

| Data Type | Records/Year | Storage | Update Frequency |
|-----------|-------------|---------|------------------|
| Agency Budgets | ~400 | < 1 MB | Annually |
| Contract Awards | ~10 million | ~50 GB | Daily |
| Grant Awards | ~500,000 | ~5 GB | Daily |
| Direct Payments | Variable | ~10 GB | Daily |

---

## 🔄 Data Synchronization Strategy

### Initial Load

1. **Historical Congressional Data**: Import last 3 Congress sessions (6 years)
2. **Election Results**: Import last 3 election cycles (6 years)
3. **Campaign Finance**: Import last 2 cycles (4 years)
4. **Federal Spending**: Import last fiscal year

### Incremental Updates

```java
@Scheduled(cron = "0 0 2 * * ?") // 2 AM daily
public void updateCongressionalData() {
    // Update member information
    // Import new bills since last run
    // Import new votes since last run
    // Update bill statuses
}

@Scheduled(cron = "0 0 3 * * ?") // 3 AM daily
public void updateElectionData() {
    // Update candidate filings
    // Import new campaign finance reports
    // Update election results (during election periods)
}

@Scheduled(cron = "0 0 4 * * ?") // 4 AM daily
public void updateSpendingData() {
    // Import new federal spending transactions
    // Update agency budget information
}
```

### Real-time Updates (During Active Periods)

During active legislative periods or elections:
- Poll Congress.gov API every 15 minutes for vote updates
- Poll ProPublica API every hour for member updates
- Poll FEC API every 6 hours during filing deadlines

---

## 🛠️ Implementation Example

### Sample Service Implementation

```java
@Service
public class CongressMemberServiceImpl implements CongressMemberService {
    
    private final RestTemplate restTemplate;
    
    @Value("${congress.api.key}")
    private String apiKey;
    
    @Value("${congress.api.base.url}")
    private String baseUrl;
    
    @Override
    public List<CongressMember> getCurrentMembers(String chamber) {
        String url = baseUrl + "/member?currentMember=true";
        
        HttpHeaders headers = new HttpHeaders();
        headers.set("X-API-Key", apiKey);
        headers.setAccept(Collections.singletonList(MediaType.APPLICATION_JSON));
        
        HttpEntity<String> entity = new HttpEntity<>(headers);
        
        ResponseEntity<CongressMemberResponse> response = 
            restTemplate.exchange(url, HttpMethod.GET, entity, CongressMemberResponse.class);
        
        return transformToInternalModel(response.getBody());
    }
    
    private List<CongressMember> transformToInternalModel(CongressMemberResponse response) {
        // Transform API response to internal data model
        return response.getMembers().stream()
            .map(this::mapToCongressMember)
            .collect(Collectors.toList());
    }
}
```

---

## 📚 Additional Resources

### Official Documentation
- [Congress.gov API Documentation](https://api.congress.gov/)
- [ProPublica Congress API Documentation](https://projects.propublica.org/api-docs/congress-api/)
- [FEC API Documentation](https://api.open.fec.gov/developers/)
- [USAspending.gov API Documentation](https://api.usaspending.gov/)

### Data Standards
- [Legislative Markup (USLM)](https://github.com/usgpo/uslm)
- [Federal Program Inventory](https://www.performance.gov/federalprograminventory/)
- [Congressional Identifiers](https://bioguide.congress.gov/)

### Community Resources
- [Open States Project](https://openstates.org/) - State-level legislative data
- [GovTrack.us](https://www.govtrack.us/) - Alternative congressional data source
- [Sunlight Foundation Archives](https://sunlightfoundation.com/) - Historical transparency data

---

**Document Status:** Reference Implementation Guide  
**Last Updated:** 2025-12-28  
**Maintained By:** Development Team
