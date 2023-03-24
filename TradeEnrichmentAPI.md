### Microservice architecture background for student's understanding

- In old days, large projects were used to be huge monolithic applications. 
- In modern era these monolithic applications are dismantled into over the time into smaller applications



#### Wikipedia's definition of microservices

A microservice architecture – a variant of the service-oriented architecture structural style – is an architectural 
pattern that arranges an application as a collection of loosely coupled, fine-grained services, communicating through
lightweight protocols.


#### aws' definition of microservices

**https://aws.amazon.com/microservices/**

Microservices are an architectural and organizational approach to software development where software is composed of
small independent services that communicate over well-defined APIs. These services are owned by small, self-contained 
teams.
Microservices architectures make applications easier to scale and faster to develop, enabling innovation and 
accelerating time-to-market for new features.


#### Monolithic vs. Microservices Architecture

With monolithic architectures, all processes are tightly coupled and run as a single service. This means that if one
process of the application experiences a spike in demand, the entire architecture must be scaled. Adding or improving a
monolithic application’s features becomes more complex as the code base grows. This complexity limits experimentation
and makes it difficult to implement new ideas. Monolithic architectures add risk for application availability because
many dependent and tightly coupled processes increase the impact of a single process failure.

With a `microservices` architecture, an application is built as independent components that run each application process
as a service. These services communicate via a well-defined interface using lightweight APIs. Services are built for
business capabilities and each service performs a single function. Because they are independently run, each service can
be updated, deployed, and scaled to meet demand for specific functions of an application.

#### Project Name: `TradeEnrichmentAPI`

# Storyline

Our project is microservice architecture containing multiple REST APIs. The project is aimed to solve real time trade
validation for the investment agencies. The end users of our application are traders associated with large investment
management firms. The traders need to perform trade validation based on certain set of rules coming from variety of the
policymaking bodies. For example, in our project we have the following categories of rules -

- client jurisdiction mandate (CJM)  [Table name : `CJMRules`]

  - Any trade must be validated against set of rules prescribed state authorized body.
  - For example Indian investment agencies need to comply against the rules of `SEBI`.
  
- trader mandate (TM)  [Table name : `TMRules`]

  - Investment agencies do allotment of traders based on client category. 
  - Firm may choose particular experienced trader to cater particular category of client.

- exclusions (EX)  [Table name : `EXRules`]

    - These are strict rules and any trade that violates these rules is strictly prohibited.
  
- client exclusion (CE)  [Table name : `CERules`]

    - Clients have opted to apply certain set of rules on themselves.
    - For instance certain client may not want to invest in stocks from certain type of industry.
    - In such case, there should be alert/ warning raised by the system before trader proceeds further.

I have worked on following APIs

    - RuleInceptionAPI
           - API to capture and store rules from policymaking bodies
           - We can perform all kinds of CRUD operations on these rules using `RuleInceptionAPI`
           - API is built using `Django/Flask` and we have specific endpoints for each rule (rule is resource here)
           - As DB backend we are using `MongoDB` 
           - Our rules data becomes unstructured since our trade rule attributes can be very customized per client
           - That is the reason we have chosen `MongoDB` over traditional `RDBMS` databases.
    - RulesAPI
           - This application validates a trade against EX, CE, TM, CJM rules
           - It needs to flag a requested trade in 3 statuses - valid, invalid, warning.
           - This application is time critical
           - As per business requirement the turn around response time of this API needs to be less than 30 seconds.
           - Hence we have chosen to build this application using `FastAPI` (for better performance)
           - Also at the backend, we have used `mongoDB` for better performance


## Expected interview questions for above story line

```markdown
1. Where do you source your data from? 
2. What is source of your data?
3. Where do you get these rules from?
    - We have a data engineering team which collects CJM rules from firm's central knowledge base.
    - We do not take care of sourcing CJM rules.
    - For the CE, TM, EX rules we have dummy rules created through automation scripts.
    - We keep adding additional rules to test various business scenarios via our `rules-inception API`.
    - For example, we want to change `experience tenure` of platinum category trader. We can do so by sending a PATCH 
      request to existing record in `TMRules` table.
```
-------------------------------------------------------------------------------------
```markdown
1. Why mongoDB?
2. Why not RDBMS?
3. Why not MySQL?
    - Actually, this decision had been already taken by our project architect and other senior colleagues before me 
      on-boarding to the project.
    - But to answer your question, our data is unstructured in nature. Policies keep changing and so the attributes 
      associated with rules keep changing as well. We did not want to adhere to certain structure of rules because of 
      such dynamic requirements. So `NoSQL` database was the best choice for us. We opted `mongoDB`.

```
-------------------------------------------------------------------------------------

```markdown
1. Why FastAPI?
2. Why not Django/Flask?
    - Actually, this decision had been already taken by our project architect and other senior colleagues before me 
      on-boarding to the project.
    - But to answer your question, our team was already using `FastAPI` for other APIs. So there was in house knowledge
      about the `FastAPI` web framework.
    - At present date, `FastAPI` is the fastest web framework available in Python web framework ecosystem.
    - `RulesAPI` is a REST API (aka microservice) that is time critical
    - As per business requirement, `RulesAPI` had SLA to provide response in 30 seconds (SLA = Service Level Agreement)
    
```
--------------------------------------------------------------------------------------

```markdown
1. What was the volume of data?
2. How many rules were there in total?
3. What were rules like? Can you give some examples?
    - Actually, I cannot precisely say volume, but we had roughly about 30K records approx in CJM rules,
      1 K records approx in CE rules, 500 records in EX rules.
    - Our product team (business analyst) keeps gathering requirements and new set of rules.  
    - To be honest, the project is still it's inception phase and over the time the volume of rules would surely be in
      millions.
    - So considering the future scope of project, we have chosen `MongoDB` to store the rules.  
    
    - Rules can be such as follows -
```
```python
import rule_engine

# match a literal first name and applying a regex to the email
rule = rule_engine.Rule(
    'first_name == "Luke" and email =~ ".*@rebels.org$"'
) # => <Rule text='first_name == "Luke" and email =~ ".*@rebels.org$"' >

rule.matches({
    'first_name': 'Luke', 'last_name': 'Skywalker', 'email': 'luke@rebels.org'
}) # => True

rule.matches({
   'first_name': 'Darth', 'last_name': 'Vader', 'email': 'dvader@empire.net'
}) # => False
```

--------------------------------------------------------------------------------------

```markdown
1. What used to happen after trade is marked as valid/invalid?
2. What would traders do once trade has been validated?

    - Each trade goes through its life cycle like `trade validation -> actual trade -> post trade audit` etc.
    - Our product team (business analyst) keeps gathering requirements and new set of rules.  
    - To be honest, the project is still it's inception phase and over the time the volume of rules would surely be in
      millions.
    - So considering the future scope of project, we have chosen `MongoDB` to store the rules
    
```

--------------------------------------------------------------------------------------

```markdown
1. What other APIs did you have in the project?

     - We had other APIs to perform other parts of processes during trade lifecycle, but I did not get opportunity to 
      work on them.
    - To give you an example, we had one microservice dedicated for `Identity and Access Management` (`IDAM`)
    - This `IDAM` API was consumed by all other APIs to grab JWT access token

```


--------------------------------------------------------------------------------------

```markdown
1. What was your team composition like?
2. What is your experience/ role in the project?

     - We had separate and dedicated data engineering team who would do data sourcing part for us
     - In my development team (scrum team), we were 8 people 
       (3 senior developers, 1 architect, 1 DB admin, 1 devops, 2 mid/junior developer ).
     - Our scrum team was headed by our manager directly
     - Daily standup was conducted by scrum master (which was our manager himself)
     - Our product team also used to join daily scrum
     - We had adopted agile delivery method for this project (although not all agile metrics were strictly followed as 
      the project requirements were still taking shape)
     - We had started this project about a year ago. It was only `RulesInceptionAPI` for the first 6 months.
     - `RulesAPI` development has started since past 7-8 months only.
     - I was one of the Python developers in the team
     
```

--------------------------------------------------------------------------------------
