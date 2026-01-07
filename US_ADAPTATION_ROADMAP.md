# 🇺🇸 US Adaptation Implementation Roadmap

**Version:** 1.0  
**Date:** 2025-12-28  
**Status:** Planning Phase

---

## 📋 Executive Summary

This document outlines the technical implementation roadmap for adapting the Citizen Intelligence Agency platform from Swedish to US political intelligence monitoring. The adaptation requires changes across documentation, data models, external service integrations, database schemas, and application code.

### Scope of Changes

| Category | Estimated Changes | Complexity | Priority |
|----------|------------------|------------|----------|
| Documentation | 50+ files | Low | ✅ In Progress |
| External Service APIs | 5 modules | High | 🔴 Critical |
| Data Models | 15+ entity models | High | 🔴 Critical |
| Database Schema | 85+ views, 93 tables | Very High | 🔴 Critical |
| Application Code | 1,372 Java files | Medium | 🟡 Medium |
| Configuration | 10+ property files | Low | 🟢 Low |

---

## 🎯 Phase 1: Documentation Updates (COMPLETED)

### Completed Items
- ✅ README.md - Updated mission, data sources, focus areas
- ✅ ARCHITECTURE.md - Updated C4 diagrams with US APIs
- ✅ DATA_MODEL.md - Updated overview text
- ✅ PRODUCT_SUMMARY.md - Updated OSINT data sources
- ✅ dashboard.md - US Congressional context
- ✅ MINDMAP.md - Congress vs Parliament references
- ✅ BUSINESS_PRODUCT_DOCUMENT.md - US market data
- ✅ Removed dashboard_sv.md

### Remaining Documentation Updates
- [ ] DATA_ANALYSIS_INTOP_OSINT.md - Update analytical framework examples
- [ ] DATABASE_VIEW_INTELLIGENCE_CATALOG.md - Update view descriptions
- [ ] RISK_RULES_INTOP_OSINT.md - Update risk rule context
- [ ] CHANGELOG_INTELLIGENCE.md - Update data source references
- [ ] FLOWCHART.md - Update data flow diagrams
- [ ] SWOT.md - Update strategic analysis for US market
- [ ] FUTURE_*.md documents - Update future vision docs
- [ ] Java package documentation - Update package-info.java files

---

## 🔌 Phase 2: US Data Source Integration

### 2.1 Identify US API Equivalents

#### Congressional Data Sources

| Swedish Source | US Equivalent | API Documentation | Features |
|---------------|---------------|-------------------|----------|
| **Riksdagen API** | **Congress.gov API** | [API Docs](https://www.congress.gov/about/data) | Bills, votes, members, committees |
| **Riksdagen API** | **ProPublica Congress API** | [API Docs](https://www.propublica.org/datastore/api/propublica-congress-api) | Member data, votes, bills, statements |
| **Swedish Election Authority** | **Federal Election Commission API** | [API Docs](https://api.open.fec.gov/developers/) | Campaign finance, election results |
| **ESV (Swedish Financial Authority)** | **USAspending.gov API** | [API Docs](https://api.usaspending.gov/) | Federal spending, contracts, grants |
| **World Bank** | **World Bank API** | [API Docs](https://datahelpdesk.worldbank.org/knowledgebase/articles/889392) | Economic indicators (unchanged) |

#### Additional US Data Sources

| Data Type | Source | API/Format | Use Case |
|-----------|--------|------------|----------|
| **Legislative Text** | GovInfo API | XML/JSON | Bill full text and amendments |
| **Voting Records** | Congressional Record | XML | Floor proceedings and votes |
| **Committee Data** | House/Senate Clerk APIs | JSON | Committee assignments, hearings |
| **Financial Disclosure** | House Ethics & Senate EFDS | PDF/XML | Personal financial disclosures |
| **Lobbying Data** | Senate Lobbying Disclosure | XML | Lobbying activity and expenditures |

### 2.2 API Capability Mapping

#### Congress.gov API
```
Capabilities:
- Member profiles (House & Senate)
- Bill tracking and status
- Committee information
- Voting records
- Amendments
- Treaty information
- Nominations

Rate Limits: API key required, reasonable rate limits
Authentication: API key (free registration)
Data Format: JSON/XML
Update Frequency: Daily
```

#### ProPublica Congress API
```
Capabilities:
- Member biographical data
- Voting records and positions
- Bill sponsorship and cosponsorship
- Committee assignments
- Floor statements
- Office expenses
- Member effectiveness metrics

Rate Limits: 5,000 requests/day (free tier)
Authentication: API key required
Data Format: JSON
Update Frequency: Daily
```

#### Federal Election Commission (FEC) API
```
Capabilities:
- Campaign finance data
- Candidate information
- Committee filings
- Election results
- Disbursements and receipts
- Independent expenditures

Rate Limits: 1,000 requests/hour (API key)
Authentication: API key (free)
Data Format: JSON
Update Frequency: Daily (filings: real-time)
```

#### USAspending.gov API
```
Capabilities:
- Federal spending by agency
- Contract awards
- Grant awards
- Loan programs
- Direct payments
- Budget authority

Rate Limits: Reasonable, no strict limits
Authentication: Not required for most endpoints
Data Format: JSON/CSV
Update Frequency: Daily
```

### 2.3 Module Refactoring Plan

#### External Service Module Structure

Current Swedish modules:
```
service.external.riksdagen/
  - RiksdagenDocumentApi
  - RiksdagenPersonApi
  - RiksdagenBallotApi
  - RiksdagenCommitteeProposalApi

service.external.val/
  - Election data services

service.external.esv/
  - Financial data services
```

Proposed US module structure:
```
service.external.congress/
  - CongressApi (Congress.gov integration)
  - ProPublicaCongressApi
  - BillTrackingService
  - MemberService
  - VotingService
  - CommitteeService

service.external.fec/
  - CampaignFinanceApi
  - ElectionResultsApi
  - CandidateService
  - CommitteeFilingService

service.external.usaspending/
  - FederalSpendingApi
  - ContractAwardService
  - GrantService

service.external.worldbank/
  - (Keep existing - already country-agnostic)
```

### 2.4 Data Model Migration

#### Entity Model Changes

Current Riksdagen entities need US Congress equivalents:

| Swedish Model | US Model | Notes |
|--------------|----------|-------|
| `model.external.riksdagen.person` | `model.external.congress.member` | Congressional member data |
| `model.external.riksdagen.votering` | `model.external.congress.vote` | Voting records |
| `model.external.riksdagen.dokumentlista` | `model.external.congress.bill` | Bill and document tracking |
| `model.external.riksdagen.dokumentstatus` | `model.external.congress.billstatus` | Bill lifecycle status |
| `model.external.val.partier` | `model.external.fec.party` | Political party data |
| `model.external.val.riksdagsvalkrets` | `model.external.fec.district` | Congressional districts |

---

## 🗄️ Phase 3: Database Schema Migration

### 3.1 Schema Impact Analysis

The database currently contains:
- **93 tables** - Political entity data storage
- **57 regular views** - Query abstractions
- **28 materialized views** - Performance optimizations

Key schema areas requiring updates:

#### Tables Requiring Changes
```sql
-- Parliament/Congress tables
riksdagen_person → congress_member
riksdagen_vote_data → congress_vote_data
riksdagen_document_element → congress_bill
riksdagen_committee_proposal → congress_committee_proposal

-- Election tables
val_party → fec_party
val_election_result → fec_election_result
val_riksdagsvalkrets → fec_congressional_district

-- Government body tables
esv_government_body → federal_agency
esv_government_body_annual_summary → federal_agency_annual_summary
```

#### Views Requiring Updates

All 85 views with `riksdagen` or `val` prefixes need renaming and logic updates:

```sql
-- Intelligence views
view_riksdagen_intelligence_dashboard → view_congress_intelligence_dashboard
view_riksdagen_party_momentum_analysis → view_party_momentum_analysis
view_riksdagen_politician_influence_metrics → view_congress_member_influence_metrics

-- Summary views
view_riksdagen_politician_summary → view_congress_member_summary
view_riksdagen_party_summary → view_party_summary
view_riksdagen_vote_data_ballot_summary → view_congress_vote_summary
```

### 3.2 Migration Strategy

#### Option 1: Parallel Schema (Recommended for Development)
```
Pros:
- Keep existing Swedish data for reference
- Develop and test US schema alongside
- Easier rollback
- Can maintain both for comparison

Cons:
- Requires more database space
- More complex deployment
- Need to maintain both during transition
```

#### Option 2: In-Place Migration
```
Pros:
- Single schema to maintain
- Cleaner long-term solution
- Less storage requirements

Cons:
- Requires complete data export/reimport
- No rollback capability
- More risky deployment
```

**Recommendation:** Use Option 1 (Parallel Schema) initially, then deprecate Swedish schema once US implementation is stable.

### 3.3 Liquibase Changelog Requirements

New Liquibase changesets needed:

```xml
<!-- Create US Congressional tables -->
<changeSet id="create-congress-member-table" author="migration">
  <createTable tableName="congress_member">
    <column name="id" type="varchar(255)">
      <constraints primaryKey="true"/>
    </column>
    <column name="first_name" type="varchar(255)"/>
    <column name="last_name" type="varchar(255)"/>
    <column name="state" type="varchar(2)"/>
    <column name="district" type="int"/>
    <column name="party" type="varchar(50)"/>
    <column name="chamber" type="varchar(10)"/>
    <!-- Additional columns -->
  </createTable>
</changeSet>

<!-- Create US voting tables -->
<changeSet id="create-congress-vote-table" author="migration">
  <!-- Vote data structure -->
</changeSet>

<!-- Create US bill tracking tables -->
<changeSet id="create-congress-bill-table" author="migration">
  <!-- Bill data structure -->
</changeSet>
```

---

## 💻 Phase 4: Application Code Updates

### 4.1 Java Package Refactoring

Current package structure uses Swedish naming:

```java
com.hack23.cia.model.external.riksdagen.*
com.hack23.cia.model.external.val.*
com.hack23.cia.service.external.riksdagen.*
com.hack23.cia.service.external.val.*
```

Proposed US package structure:

```java
com.hack23.cia.model.external.congress.*
com.hack23.cia.model.external.fec.*
com.hack23.cia.service.external.congress.*
com.hack23.cia.service.external.fec.*
com.hack23.cia.service.external.usaspending.*
```

### 4.2 Service Implementation Requirements

Each new external service module needs:

#### Congress.gov Service Implementation
```java
@Service
public class CongressApiImpl implements CongressApi {
    
    @Value("${congress.api.key}")
    private String apiKey;
    
    @Value("${congress.api.base.url}")
    private String baseUrl;
    
    @Override
    public List<CongressMember> getMembers(int congress, String chamber) {
        // Implementation
    }
    
    @Override
    public List<Bill> getBills(int congress, String billType) {
        // Implementation
    }
    
    @Override
    public List<Vote> getVotes(int congress, String chamber) {
        // Implementation
    }
}
```

#### ProPublica API Service
```java
@Service
public class ProPublicaCongressApiImpl implements ProPublicaCongressApi {
    
    @Value("${propublica.api.key}")
    private String apiKey;
    
    @Override
    public MemberDetails getMemberDetails(String memberId) {
        // Implementation
    }
    
    @Override
    public List<VotePosition> getMemberVotes(String memberId) {
        // Implementation
    }
}
```

#### FEC API Service
```java
@Service
public class FecApiImpl implements FecApi {
    
    @Value("${fec.api.key}")
    private String apiKey;
    
    @Override
    public List<CampaignFinanceRecord> getCampaignFinance(String candidateId) {
        // Implementation
    }
    
    @Override
    public List<ElectionResult> getElectionResults(int year, String state) {
        // Implementation
    }
}
```

### 4.3 Data Import Scheduling

Update Spring Integration flows for US data sources:

```xml
<!-- Congress.gov data import -->
<int:poller id="congressDataPoller" 
            cron="0 0 2 * * ?" />

<!-- ProPublica data import -->
<int:poller id="propublicaDataPoller" 
            cron="0 0 3 * * ?" />

<!-- FEC data import -->
<int:poller id="fecDataPoller" 
            cron="0 0 4 * * ?" />

<!-- USAspending data import -->
<int:poller id="usaspendingDataPoller" 
            cron="0 0 5 * * ?" />
```

---

## ⚙️ Phase 5: Configuration Updates

### 5.1 Property Files

Update configuration in `/opt/cia/webapps/cia/WEB-INF/`:

```properties
# US API Configuration
congress.api.key=${CONGRESS_API_KEY}
congress.api.base.url=https://api.congress.gov/v3

propublica.api.key=${PROPUBLICA_API_KEY}
propublica.api.base.url=https://api.propublica.org/congress/v1

fec.api.key=${FEC_API_KEY}
fec.api.base.url=https://api.open.fec.gov/v1

usaspending.api.base.url=https://api.usaspending.gov/api/v2

# Data import settings
data.import.congress.enabled=true
data.import.propublica.enabled=true
data.import.fec.enabled=true
data.import.usaspending.enabled=true

# Legacy Swedish sources (disabled)
data.import.riksdagen.enabled=false
data.import.val.enabled=false
data.import.esv.enabled=false
```

### 5.2 Agent Configuration

Update JMS agent settings:

```properties
# Congress data import agents
com.hack23.cia.service.component.agent.impl.congress.member.import=true
com.hack23.cia.service.component.agent.impl.congress.bill.import=true
com.hack23.cia.service.component.agent.impl.congress.vote.import=true

# FEC data import agents
com.hack23.cia.service.component.agent.impl.fec.finance.import=true
com.hack23.cia.service.component.agent.impl.fec.election.import=true
```

---

## 🧪 Phase 6: Testing Strategy

### 6.1 Unit Testing

- [ ] Create unit tests for new API integration services
- [ ] Test data transformation logic
- [ ] Test error handling and rate limiting
- [ ] Validate data model mappings

### 6.2 Integration Testing

- [ ] Test API connectivity with real endpoints
- [ ] Validate data import workflows
- [ ] Test database schema migrations
- [ ] Verify view calculations with US data

### 6.3 System Testing

- [ ] End-to-end data flow validation
- [ ] Performance testing with realistic data volumes
- [ ] UI testing with US political data
- [ ] Security testing of new API integrations

---

## 📊 Phase 7: Data Quality Validation

### 7.1 Data Completeness Checks

```sql
-- Validate Congress member data
SELECT COUNT(*) as member_count,
       chamber,
       state
FROM congress_member
GROUP BY chamber, state;

-- Validate voting data
SELECT COUNT(*) as vote_count,
       chamber,
       DATE_TRUNC('month', vote_date) as month
FROM congress_vote_data
GROUP BY chamber, month;

-- Validate bill data
SELECT COUNT(*) as bill_count,
       bill_type,
       congress
FROM congress_bill
GROUP BY bill_type, congress;
```

### 7.2 Data Accuracy Validation

- Compare member counts against official House/Senate records
- Validate party affiliations
- Cross-check voting records with official sources
- Verify bill counts per Congress session

---

## 🚀 Phase 8: Deployment Strategy

### 8.1 Development Environment

1. Set up US API keys (development tier)
2. Deploy parallel schema
3. Import sample US data
4. Test UI with US data
5. Validate analytics and risk rules

### 8.2 Staging Environment

1. Full schema deployment
2. Import historical data (last 2 Congress sessions)
3. Performance testing
4. Security audit
5. User acceptance testing

### 8.3 Production Deployment

1. Database migration (parallel schema approach)
2. API integration deployment
3. Data import job scheduling
4. Monitoring and alerting setup
5. Gradual traffic migration

---

## 📅 Estimated Timeline

| Phase | Duration | Dependencies | Resources |
|-------|----------|--------------|-----------|
| **Phase 1: Documentation** | 1 week | None | 1 developer |
| **Phase 2: API Integration** | 4-6 weeks | API keys, testing accounts | 2 developers |
| **Phase 3: Database Schema** | 3-4 weeks | Phase 2 completion | 1 DBA, 1 developer |
| **Phase 4: Application Code** | 6-8 weeks | Phase 2, 3 | 2-3 developers |
| **Phase 5: Configuration** | 1 week | Phase 4 | 1 developer |
| **Phase 6: Testing** | 3-4 weeks | Phase 4, 5 | 2 QA engineers |
| **Phase 7: Data Validation** | 2 weeks | Phase 6 | 1 data analyst |
| **Phase 8: Deployment** | 2-3 weeks | All phases | DevOps team |

**Total Estimated Duration:** 22-32 weeks (5.5-8 months)

---

## 🎯 Success Criteria

### Technical Metrics
- [ ] All 93 tables migrated to US schema
- [ ] All 85 views operational with US data
- [ ] API integration tests passing at >95%
- [ ] Database query performance within 10% of Swedish baseline
- [ ] Zero data loss during migration

### Functional Metrics
- [ ] Congressional member data 100% accurate
- [ ] Voting records complete for last 2 Congress sessions
- [ ] Campaign finance data current within 24 hours
- [ ] Federal spending data current within 24 hours
- [ ] All dashboards displaying US data correctly

### Quality Metrics
- [ ] Unit test coverage >80%
- [ ] Integration test coverage >70%
- [ ] Zero critical security vulnerabilities
- [ ] API rate limit compliance 100%
- [ ] Data quality score >95%

---

## 🚧 Risks and Mitigation

### Technical Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| API rate limits restrict data volume | High | Medium | Implement caching, use multiple APIs |
| Data format incompatibility | High | Medium | Extensive testing, flexible parsers |
| Schema migration complexity | High | Medium | Parallel schema approach, rollback plan |
| Performance degradation | Medium | Low | Load testing, query optimization |

### Business Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| API access restrictions/costs | High | Low | Establish relationships, budget for paid tiers |
| Data accuracy issues | High | Medium | Validation framework, cross-checking |
| Legal/compliance concerns | Medium | Low | Legal review, terms of service compliance |

---

## 📚 References

### API Documentation
- [Congress.gov API Documentation](https://www.congress.gov/about/data)
- [ProPublica Congress API Documentation](https://www.propublica.org/datastore/api/propublica-congress-api)
- [FEC API Documentation](https://api.open.fec.gov/developers/)
- [USAspending.gov API Documentation](https://api.usaspending.gov/)

### US Political Data Standards
- [United States Legislative Markup (USLM)](https://github.com/usgpo/uslm)
- [Federal Information Processing Standards (FIPS)](https://www.nist.gov/standardsgov/compliance-faqs-federal-information-processing-standards-fips)
- [Congressional District Codes](https://www.census.gov/programs-surveys/geography/about/glossary.html)

### Project Documentation
- [Architecture Documentation](ARCHITECTURE.md)
- [Data Model Documentation](DATA_MODEL.md)
- [Database Schema Maintenance](service.data.impl/README-SCHEMA-MAINTENANCE.md)

---

## 📞 Contact & Support

For questions about this roadmap:
- **Project Lead**: See [CODEOWNERS](CODEOWNERS)
- **Technical Questions**: Create an issue with label `us-adaptation`
- **Architecture Review**: Tag architecture team in PR

---

**Document Status:** Draft - Planning Phase  
**Last Updated:** 2025-12-28  
**Next Review:** After Phase 1 completion
