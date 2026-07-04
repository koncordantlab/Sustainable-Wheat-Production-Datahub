# GRIP Datahub Competency Questions

Ready-to-run SPARQL for the GRIP Datahub, the federated wheat knowledge graph deployed in GraphDB. Each query is self-contained: paste it into the GraphDB SPARQL editor and run it, with the prefixes included in every query.


## Namespace reference
```
nutr:  https://gwsp.cs.ksu.edu/nutrient-management/nutrient/
ncyc:  https://gwsp.cs.ksu.edu/nutrient-management/nitrogen/
san:   https://gwsp.cs.ksu.edu/nutrient-management/soil-available-nitrogen/
fert:  https://gwsp.cs.ksu.edu/nutrient-management/fertilization/
env:   https://gwsp.cs.ksu.edu/nutrient-management/environment/
obs:   https://gwsp.cs.ksu.edu/nutrient-management/observation/
ndata: https://gwsp.cs.ksu.edu/nutrient-data/      (nutrient data individuals)
ddata: https://gwsp.cs.ksu.edu/disease-data/       (disease data individuals)
dm:    https://gwsp.cs.ksu.edu/disease-management/
path:  https://gwsp.cs.ksu.edu/disease-management/pathogen/
dis:   https://gwsp.cs.ksu.edu/disease-management/disease/
mgt:   https://gwsp.cs.ksu.edu/disease-management/management/
```

# Part A. Nutrient Management

## A.1 Schema-level

### N1. Which elements are primary mineral, secondary mineral, and micronutrients?
```sparql
PREFIX rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX nutr: <https://gwsp.cs.ksu.edu/nutrient-management/nutrient/>
SELECT ?category ?element ?label WHERE {
  VALUES ?category { nutr:Primary_Mineral_Nutrient
                     nutr:Secondary_Mineral_Nutrient
                     nutr:Micronutrient }
  ?element rdf:type ?category .
  OPTIONAL { ?element rdfs:label ?label }
}
ORDER BY ?category ?element
```

### N2. What nutrient forms are represented for nitrogen?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX nutr: <https://gwsp.cs.ksu.edu/nutrient-management/nutrient/>
SELECT ?form ?label WHERE {
  ?form nutr:is_form_of nutr:Nitrogen .
  OPTIONAL { ?form rdfs:label ?label }
}
```
If empty, list all forms instead: `?form rdf:type nutr:Nutrient_Form .`

### N3. What components constitute the nitrogen balance (supply, demand, transformations, losses)?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX ncyc: <https://gwsp.cs.ksu.edu/nutrient-management/nitrogen/>
SELECT ?relation ?component ?label WHERE {
  ?relation rdfs:domain ncyc:Nitrogen_Balance ;
            rdfs:range  ?component .
  OPTIONAL { ?component rdfs:label ?label }
}
```

### N4. Which processes are nitrogen transformations, and which are loss pathways?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX ncyc: <https://gwsp.cs.ksu.edu/nutrient-management/nitrogen/>
SELECT ?kind ?process ?label WHERE {
  { ?process rdfs:subClassOf ncyc:Nitrogen_Transformation .
    BIND("transformation" AS ?kind) }
  UNION
  { ?process rdfs:subClassOf ncyc:Nitrogen_Loss .
    BIND("loss" AS ?kind) }
  OPTIONAL { ?process rdfs:label ?label }
}
ORDER BY ?kind ?process
```

### N5. What factors contribute to the soil available nitrogen pool?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl:  <http://www.w3.org/2002/07/owl#>
PREFIX ncyc: <https://gwsp.cs.ksu.edu/nutrient-management/nitrogen/>
SELECT ?direction ?process ?label WHERE {
  ncyc:Soil_Available_Nitrogen rdfs:subClassOf ?r .
  ?r owl:onProperty ?p ; owl:someValuesFrom ?process .
  VALUES (?p ?direction) {
    (ncyc:supplied_through "supply")
    (ncyc:depleted_by      "depletion")
  }
  OPTIONAL { ?process rdfs:label ?label }
}
```
If empty, the relations may be direct triples:
`ncyc:Soil_Available_Nitrogen ncyc:supplied_through ?process .`

### N6. What metrics are defined for nutrient use efficiency?
```sparql
PREFIX rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX nutr: <https://gwsp.cs.ksu.edu/nutrient-management/nutrient/>
SELECT ?metric ?label WHERE {
  ?metric rdf:type ?c .
  ?c rdfs:subClassOf* nutr:Nutrient_Use_Efficiency .
  OPTIONAL { ?metric rdfs:label ?label }
}
```

### N7. Which crop responses are used to evaluate nutrient management?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX nutr: <https://gwsp.cs.ksu.edu/nutrient-management/nutrient/>
SELECT ?responseClass ?label WHERE {
  ?responseClass rdfs:subClassOf nutr:Crop_Response .
  OPTIONAL { ?responseClass rdfs:label ?label }
}
```

### N8. Which fertilizer sources are represented, and what is the guaranteed analysis of each?
```sparql
PREFIX rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX fert: <https://gwsp.cs.ksu.edu/nutrient-management/fertilization/>
SELECT ?fertilizer ?label ?N ?P2O5 ?K2O ?S WHERE {
  ?fertilizer rdf:type ?c .
  ?c rdfs:subClassOf* fert:Fertilizer_Source .
  OPTIONAL { ?fertilizer rdfs:label ?label }
  OPTIONAL { ?fertilizer fert:has_N_percent    ?N }
  OPTIONAL { ?fertilizer fert:has_P2O5_percent ?P2O5 }
  OPTIONAL { ?fertilizer fert:has_K2O_percent  ?K2O }
  OPTIONAL { ?fertilizer fert:has_S_percent    ?S }
}
```

### N9. What management practices are associated with fertilization?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX fert: <https://gwsp.cs.ksu.edu/nutrient-management/fertilization/>
SELECT ?practice ?label WHERE {
  ?practice rdfs:subClassOf+ fert:Management_Practice .
  OPTIONAL { ?practice rdfs:label ?label }
}
ORDER BY ?practice
```

### N10. Which soil characteristics and soil texture classes are represented?
```sparql
PREFIX rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX env:  <https://gwsp.cs.ksu.edu/nutrient-management/environment/>
SELECT ?kind ?item ?label WHERE {
  { ?item rdfs:subClassOf+ env:Soil_Component .
    BIND("soil component (class)" AS ?kind) }
  UNION
  { ?item rdf:type env:Soil_Texture .
    BIND("soil texture (individual)" AS ?kind) }
  OPTIONAL { ?item rdfs:label ?label }
}
ORDER BY ?kind ?item
```

## A.2 Data-level

### N11. For each variety and location, the maximum protein and test weight, top 50 by protein.
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obs:  <https://gwsp.cs.ksu.edu/nutrient-management/observation/>
SELECT ?variety ?location (MAX(?protein) AS ?maxProtein) (MAX(?tw) AS ?maxTestWeight)
WHERE {
  ?o a obs:Plot_Observation ;
     obs:observed_cultivar     ?cv ;
     obs:observed_at_site_year ?sye ;
     obs:has_protein_pct_dry_basis ?protein ;
     obs:has_test_weight_lbs_per_bu ?tw .
  ?cv  rdfs:label ?variety .
  ?sye obs:has_location ?loc .
  ?loc rdfs:label ?location .
}
GROUP BY ?variety ?location
ORDER BY DESC(?maxProtein)
LIMIT 50
```

### N12. What is the average yield of each variety across all trials?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obs:  <https://gwsp.cs.ksu.edu/nutrient-management/observation/>
SELECT ?variety (AVG(?y) AS ?avgYield) (COUNT(?o) AS ?n) WHERE {
  ?o a obs:Plot_Observation ;
     obs:observed_cultivar ?cv ;
     obs:has_yield_lbs_per_acre_13pct ?y .
  ?cv rdfs:label ?variety .
}
GROUP BY ?variety
ORDER BY DESC(?avgYield)
```

### N13. For the variety Zenda, how do yield and protein vary across trials and locations?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obs:  <https://gwsp.cs.ksu.edu/nutrient-management/observation/>
SELECT ?trial ?location ?yield ?protein WHERE {
  ?o a obs:Plot_Observation ;
     obs:observed_cultivar     ?cv ;
     obs:observed_in_trial     ?t ;
     obs:observed_at_site_year ?sye .
  ?cv rdfs:label ?variety .
  FILTER( LCASE(STR(?variety)) = "zenda" )
  ?t   rdfs:label ?trial .
  ?sye obs:has_location ?loc .
  ?loc rdfs:label ?location .
  OPTIONAL { ?o obs:has_yield_lbs_per_acre_13pct ?yield }
  OPTIONAL { ?o obs:has_protein_pct_dry_basis    ?protein }
}
ORDER BY ?trial ?location
```

### N14. List the unique varieties tested across all trials.
```sparql
PREFIX rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX nutr: <https://gwsp.cs.ksu.edu/nutrient-management/nutrient/>
SELECT DISTINCT ?variety WHERE {
  ?cv rdf:type nutr:Cultivar ;
      rdfs:label ?variety .
}
ORDER BY ?variety
```

### N15. In which plots was the previous crop soybean?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obs:  <https://gwsp.cs.ksu.edu/nutrient-management/observation/>
SELECT ?o ?label WHERE {
  ?o a obs:Plot_Observation ;
     obs:observed_previous_crop ?pc .
  ?pc rdfs:label ?pcName .
  FILTER( CONTAINS( LCASE(STR(?pcName)), "soybean" ) )
  OPTIONAL { ?o rdfs:label ?label }
}
```

### N16. For each previous-crop category, the average yield of the following wheat crop.
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obs:  <https://gwsp.cs.ksu.edu/nutrient-management/observation/>
PREFIX fert: <https://gwsp.cs.ksu.edu/nutrient-management/fertilization/>
SELECT ?cat (SAMPLE(?catLabel) AS ?category)
       (AVG(?y) AS ?avgYield) (COUNT(?o) AS ?n) WHERE {
  ?o a obs:Plot_Observation ;
     obs:observed_previous_crop ?pc ;
     obs:has_yield_lbs_per_acre_13pct ?y .
  ?pc fert:has_crop_category ?cat .
  OPTIONAL { ?cat rdfs:label ?catLabel }
}
GROUP BY ?cat
ORDER BY DESC(?avgYield)
```

### N17. For each location and year, how many plots were recorded and the average yield.
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obs:  <https://gwsp.cs.ksu.edu/nutrient-management/observation/>
SELECT ?location ?year (COUNT(?o) AS ?plots) (AVG(?y) AS ?avgYield) WHERE {
  ?o a obs:Plot_Observation ;
     obs:observed_at_site_year ?sye .
  OPTIONAL { ?o obs:has_yield_lbs_per_acre_13pct ?y }
  ?sye obs:has_location ?loc ;
       obs:has_year     ?yr .
  ?loc rdfs:label ?location .
  ?yr  rdfs:label ?year .
}
GROUP BY ?location ?year
ORDER BY ?location ?year
```

### N18. Which variety achieved the highest average yield at each location?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obs:  <https://gwsp.cs.ksu.edu/nutrient-management/observation/>
SELECT ?location ?variety ?avgYield WHERE {
  {
    SELECT ?location ?variety (AVG(?y) AS ?avgYield) WHERE {
      ?o a obs:Plot_Observation ;
         obs:observed_cultivar ?cv ;
         obs:observed_at_site_year ?sye ;
         obs:has_yield_lbs_per_acre_13pct ?y .
      ?cv  rdfs:label ?variety .
      ?sye obs:has_location ?loc .
      ?loc rdfs:label ?location .
    }
    GROUP BY ?location ?variety
  }
  {
    SELECT ?location (MAX(?avgY) AS ?topYield) WHERE {
      SELECT ?location ?variety (AVG(?y) AS ?avgY) WHERE {
        ?o a obs:Plot_Observation ;
           obs:observed_cultivar ?cv ;
           obs:observed_at_site_year ?sye ;
           obs:has_yield_lbs_per_acre_13pct ?y .
        ?cv  rdfs:label ?variety .
        ?sye obs:has_location ?loc .
        ?loc rdfs:label ?location .
      }
      GROUP BY ?location ?variety
    }
    GROUP BY ?location
  }
  FILTER( ?avgYield = ?topYield )
}
ORDER BY ?location
```

### N19. For each nitrogen timing split, the average yield and protein.
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obs:  <https://gwsp.cs.ksu.edu/nutrient-management/observation/>
SELECT ?split ?label (AVG(?y) AS ?avgYield) (AVG(?p) AS ?avgProtein) (COUNT(?o) AS ?n)
WHERE {
  ?o a obs:Plot_Observation ;
     obs:observed_n_timing ?split .
  OPTIONAL { ?o obs:has_yield_lbs_per_acre_13pct ?y }
  OPTIONAL { ?o obs:has_protein_pct_dry_basis    ?p }
  OPTIONAL { ?split rdfs:label ?label }
}
GROUP BY ?split ?label
ORDER BY ?split
```

### N20. Which plots recorded grain protein above 14%?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obs:  <https://gwsp.cs.ksu.edu/nutrient-management/observation/>
SELECT ?o ?variety ?protein WHERE {
  ?o a obs:Plot_Observation ;
     obs:has_protein_pct_dry_basis ?protein ;
     obs:observed_cultivar ?cv .
  ?cv rdfs:label ?variety .
  FILTER( ?protein > 14.0 )
}
ORDER BY DESC(?protein)
```

---

# Part B. Disease Management

## B.1 Schema-level

### D1. Which pathogens are fungal, bacterial, and viral?
```sparql
PREFIX rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX path: <https://gwsp.cs.ksu.edu/disease-management/pathogen/>
SELECT ?type ?pathogen ?label WHERE {
  VALUES ?type { path:Fungal_Pathogen path:Bacterial_Pathogen path:Viral_Pathogen }
  ?pathogen rdf:type ?type .
  OPTIONAL { ?pathogen rdfs:label ?label }
}
ORDER BY ?type
```

### D2. Which diseases are classified as fungal, bacterial, and viral?
```sparql
PREFIX rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dis:  <https://gwsp.cs.ksu.edu/disease-management/disease/>
SELECT DISTINCT ?type ?disease ?label WHERE {
  VALUES ?type { dis:Fungal_Disease dis:Bacterial_Disease dis:Viral_Disease }
  ?disease rdf:type ?c .
  ?c rdfs:subClassOf* ?type .
  OPTIONAL { ?disease rdfs:label ?label }
}
ORDER BY ?type ?disease
```

### D3. For each wheat disease, what is its causal pathogen?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dis:  <https://gwsp.cs.ksu.edu/disease-management/disease/>
SELECT ?disease ?diseaseLabel ?pathogen ?pathogenLabel WHERE {
  ?disease dis:caused_by ?pathogen .
  OPTIONAL { ?disease  rdfs:label ?diseaseLabel }
  OPTIONAL { ?pathogen rdfs:label ?pathogenLabel }
}
ORDER BY ?diseaseLabel
```

### D4. For a given pathogen, which diseases does it cause? (example: Puccinia triticina)
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX path: <https://gwsp.cs.ksu.edu/disease-management/pathogen/>
PREFIX dis:  <https://gwsp.cs.ksu.edu/disease-management/disease/>
SELECT ?disease ?label WHERE {
  ?disease dis:caused_by path:Puccinia_triticina .
  OPTIONAL { ?disease rdfs:label ?label }
}
```

### D5. What three components define the disease triangle?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl:  <http://www.w3.org/2002/07/owl#>
PREFIX dis:  <https://gwsp.cs.ksu.edu/disease-management/disease/>
SELECT ?via ?component WHERE {
  dis:Disease_Triangle rdfs:subClassOf ?r .
  ?r owl:onProperty ?via ;
     owl:someValuesFrom ?component .
}
```

### D6. What assessment methods are defined for measuring disease?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dis:  <https://gwsp.cs.ksu.edu/disease-management/disease/>
SELECT ?method ?label WHERE {
  ?method rdfs:subClassOf+ dis:Disease_Assessment .
  OPTIONAL { ?method rdfs:label ?label }
}
```

### D7. What resistance levels are defined for rating cultivar response to disease?
```sparql
PREFIX rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dis:  <https://gwsp.cs.ksu.edu/disease-management/disease/>
SELECT ?level ?label WHERE {
  ?level rdf:type dis:Resistance_Level .
  OPTIONAL { ?level rdfs:label ?label }
}
```

### D8. Which strategies fall under chemical control, cultural control, and host resistance?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX mgt:  <https://gwsp.cs.ksu.edu/disease-management/management/>
SELECT ?pillar ?strategy ?label WHERE {
  VALUES ?pillar { mgt:Chemical_Control mgt:Cultural_Control mgt:Host_Resistance }
  ?strategy rdfs:subClassOf+ ?pillar .
  OPTIONAL { ?strategy rdfs:label ?label }
}
ORDER BY ?pillar
```

### D9. What attributes characterize a fungicide in the schema?
Reads the properties declared on the Fungicide class (FRAC group, active ingredient,
application timing, and the recorded datatype attributes), independent of any product
instances.
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX mgt:  <https://gwsp.cs.ksu.edu/disease-management/management/>
SELECT ?property ?label ?range WHERE {
  ?property rdfs:domain mgt:Fungicide .
  OPTIONAL { ?property rdfs:label ?label }
  OPTIONAL { ?property rdfs:range ?range }
}
ORDER BY ?property
```

### D10. What types of host resistance are distinguished?
```sparql
PREFIX rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX mgt:  <https://gwsp.cs.ksu.edu/disease-management/management/>
SELECT ?type ?label WHERE {
  ?type rdf:type mgt:Resistance_Type .
  OPTIONAL { ?type rdfs:label ?label }
}
```

## B.2 Data-level

### D11. Which fungicide products were applied, how often, and at what mean rate?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX mgt:  <https://gwsp.cs.ksu.edu/disease-management/management/>
SELECT ?product ?label (COUNT(?app) AS ?applications) (AVG(?rate) AS ?meanRate)
WHERE {
  ?app a mgt:Fungicide_Application ;
       mgt:applied_fungicide ?product .
  OPTIONAL { ?app mgt:has_application_rate ?rate }
  OPTIONAL { ?product rdfs:label ?label }
}
GROUP BY ?product ?label
ORDER BY DESC(?applications)
```

### D12. For each variety and target disease, the average recorded disease severity.
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obs:  <https://gwsp.cs.ksu.edu/nutrient-management/observation/>
PREFIX dis:  <https://gwsp.cs.ksu.edu/disease-management/disease/>
SELECT ?d ?diseaseLabel ?cv ?variety (AVG(?sev) AS ?avgSeverity) (COUNT(?a) AS ?n)
WHERE {
  ?a a dis:Disease_Assessment ;
     dis:assessed_plot ?plot ;
     dis:has_severity_percent ?sev .
  ?plot obs:has_cultivar ?cv ;
        dis:has_target_disease ?d .
  OPTIONAL { ?cv rdfs:label ?variety }
  OPTIONAL { ?d  rdfs:label ?diseaseLabel }
}
GROUP BY ?d ?diseaseLabel ?cv ?variety
ORDER BY ?d DESC(?avgSeverity)
```

### D13. How do treated and untreated plots compare on disease, mycotoxin, and yield?
A plot counts as treated if it has at least one fungicide application. Peak severity
is the maximum severity recorded across a plot's assessments. This is the query that
shows the value of the graph most directly: it joins each plot to its fungicide
applications and to its disease assessments at once, which a flat trial table does not
express.
```sparql
PREFIX obs:  <https://gwsp.cs.ksu.edu/nutrient-management/observation/>
PREFIX dis:  <https://gwsp.cs.ksu.edu/disease-management/disease/>
PREFIX mgt:  <https://gwsp.cs.ksu.edu/disease-management/management/>
SELECT ?treatment (COUNT(?plot) AS ?plots)
       (AVG(?peakSeverity) AS ?meanPeakSeverity)
       (AVG(?don)   AS ?meanDON)
       (AVG(?yield) AS ?meanYield)
       (AVG(?tw)    AS ?meanTestWeight)
WHERE {
  {
    SELECT ?plot (MAX(?sev) AS ?peakSeverity) WHERE {
      ?a dis:assessed_plot ?plot ;
         dis:has_severity_percent ?sev .
    }
    GROUP BY ?plot
  }
  OPTIONAL { ?plot dis:has_DON_ppm ?don }
  OPTIONAL { ?plot obs:has_yield_bu_per_acre ?yield }
  OPTIONAL { ?plot obs:has_test_weight_lbs_per_bu ?tw }
  BIND( IF( EXISTS { ?app mgt:applied_to_plot ?plot }, "treated", "untreated" )
        AS ?treatment )
}
GROUP BY ?treatment
```

### D14. For Fusarium head blight plots, DON alongside peak severity, per plot.
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obs:  <https://gwsp.cs.ksu.edu/nutrient-management/observation/>
PREFIX dis:  <https://gwsp.cs.ksu.edu/disease-management/disease/>
PREFIX env:  <https://gwsp.cs.ksu.edu/nutrient-management/environment/>
SELECT ?plot ?variety ?don ?peakSeverity WHERE {
  ?plot a env:Plot ;
        dis:has_target_disease dis:Fusarium_Head_Blight ;
        dis:has_DON_ppm ?don .
  OPTIONAL { ?plot obs:has_cultivar ?cv . ?cv rdfs:label ?variety }
  {
    SELECT ?plot (MAX(?sev) AS ?peakSeverity) WHERE {
      ?a dis:assessed_plot ?plot ;
         dis:has_severity_percent ?sev .
    }
    GROUP BY ?plot
  }
}
ORDER BY DESC(?don)
```

### D15. How does disease progress over the season for Fusarium head blight plots?
Each plot carries several dated assessments, so severity and incidence can be read as
a time series. To focus on one plot, add a line such as
`FILTER( ?plot = ddata:Plot_USWBI_UFT_2024_GreenHammer_B1_001 )`.
```sparql
PREFIX dis:   <https://gwsp.cs.ksu.edu/disease-management/disease/>
PREFIX env:   <https://gwsp.cs.ksu.edu/nutrient-management/environment/>
PREFIX ddata: <https://gwsp.cs.ksu.edu/disease-data/>
SELECT ?plot ?date ?severity ?incidence WHERE {
  ?plot a env:Plot ;
        dis:has_target_disease dis:Fusarium_Head_Blight .
  ?a dis:assessed_plot ?plot ;
     dis:on_date ?date .
  OPTIONAL { ?a dis:has_severity_percent ?severity }
  OPTIONAL { ?a dis:has_incidence_percent ?incidence }
}
ORDER BY ?plot ?date
```

### D16. At which growth stages were fungicides applied, and how many applications at each?
The growth stages are the Feekes individuals defined on the nutrient side and reused
here, so this query also shows the cross-module reuse in the data.
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX mgt:  <https://gwsp.cs.ksu.edu/disease-management/management/>
SELECT ?stage ?label (COUNT(?app) AS ?applications) WHERE {
  ?app a mgt:Fungicide_Application ;
       mgt:at_growth_stage ?stage .
  OPTIONAL { ?stage rdfs:label ?label }
}
GROUP BY ?stage ?label
ORDER BY DESC(?applications)
```

### D17. Does disease severity rise with crop growth stage at assessment?
Reads severity by the growth stage recorded on each assessment, again reusing the
shared Feekes individuals, this time on the assessment side.
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dis:  <https://gwsp.cs.ksu.edu/disease-management/disease/>
SELECT ?stage ?label (AVG(?sev) AS ?meanSeverity) (COUNT(?a) AS ?n) WHERE {
  ?a a dis:Disease_Assessment ;
     dis:at_growth_stage ?stage ;
     dis:has_severity_percent ?sev .
  OPTIONAL { ?stage rdfs:label ?label }
}
GROUP BY ?stage ?label
ORDER BY ?meanSeverity
```

### D18. Does peak disease severity fall as the number of fungicide applications rises?
Groups plots by how many applications each received, from zero (untreated) up to
three, and reads mean peak severity for each group. The zero group is the untreated
plots.
```sparql
PREFIX dis:  <https://gwsp.cs.ksu.edu/disease-management/disease/>
PREFIX env:  <https://gwsp.cs.ksu.edu/nutrient-management/environment/>
PREFIX mgt:  <https://gwsp.cs.ksu.edu/disease-management/management/>
SELECT ?nApplications (COUNT(?plot) AS ?plots) (AVG(?peakSeverity) AS ?meanPeakSeverity)
WHERE {
  {
    SELECT ?plot (MAX(?sev) AS ?peakSeverity) WHERE {
      ?a dis:assessed_plot ?plot ;
         dis:has_severity_percent ?sev .
    }
    GROUP BY ?plot
  }
  {
    SELECT ?plot (COUNT(?app) AS ?nApplications) WHERE {
      ?plot a env:Plot .
      OPTIONAL { ?app mgt:applied_to_plot ?plot }
    }
    GROUP BY ?plot
  }
}
GROUP BY ?nApplications
ORDER BY ?nApplications
```

### D19. For each year and target disease, plot count, mean yield, and mean peak severity.
The trial name is not a first-class individual in the current data. It appears only in
each plot's label and URI, so this summary groups by year and disease. If a trial
dimension is added to the pipeline later, it can be dropped in here.
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obs:  <https://gwsp.cs.ksu.edu/nutrient-management/observation/>
PREFIX dis:  <https://gwsp.cs.ksu.edu/disease-management/disease/>
PREFIX env:  <https://gwsp.cs.ksu.edu/nutrient-management/environment/>
SELECT ?year ?d ?diseaseLabel (COUNT(DISTINCT ?plot) AS ?plots)
       (AVG(?yield) AS ?meanYield) (AVG(?peakSeverity) AS ?meanPeakSeverity)
WHERE {
  ?plot a env:Plot ;
        obs:has_year ?year ;
        dis:has_target_disease ?d .
  OPTIONAL { ?d rdfs:label ?diseaseLabel }
  OPTIONAL { ?plot obs:has_yield_bu_per_acre ?yield }
  OPTIONAL {
    SELECT ?plot (MAX(?sev) AS ?peakSeverity) WHERE {
      ?a dis:assessed_plot ?plot ;
         dis:has_severity_percent ?sev .
    }
    GROUP BY ?plot
  }
}
GROUP BY ?year ?d ?diseaseLabel
ORDER BY ?year ?d
```

### D20. Does higher disease severity coincide with lower yield, per plot?
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obs:  <https://gwsp.cs.ksu.edu/nutrient-management/observation/>
PREFIX dis:  <https://gwsp.cs.ksu.edu/disease-management/disease/>
PREFIX env:  <https://gwsp.cs.ksu.edu/nutrient-management/environment/>
SELECT ?plot ?variety ?peakSeverity ?yield WHERE {
  ?plot a env:Plot ;
        obs:has_yield_bu_per_acre ?yield .
  OPTIONAL { ?plot obs:has_cultivar ?cv . ?cv rdfs:label ?variety }
  {
    SELECT ?plot (MAX(?sev) AS ?peakSeverity) WHERE {
      ?a dis:assessed_plot ?plot ;
         dis:has_severity_percent ?sev .
    }
    GROUP BY ?plot
  }
}
ORDER BY DESC(?peakSeverity)
```