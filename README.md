# Elections Norway


# valgresultat.no API

- https://valgresultat.no/api
- Storting
- `/[year]/st`
- Regional results
- `/[year]/st/oslo`
- `/[year]/st/rogaland`
- …
- Kommune (municipal)
- `/[year]/ko`
- Fylke (regional)
- `/[year]/fy`

# To-do

1.  Build baseline function that takes year and election type and
    outputs the overall results of the election (ideally a list of data
    frames)
    - Fill in all variables for parties
    - Include meta data
2.  Let the function accept regions, for extracting regional results
3.  Documentation

# Example

In the following code, we have set up a function that will retrieve the
overall election results for a given year and type of election.

``` r
# httr2 package for interacting with the API
library(httr2)


get_election_result <- function(year, election) {
  

  
  # Base url for API
  base_url <- "https://valgresultat.no/api"
  
  # Pasting the example
  fetch_url <- paste(base_url, year, election, sep = "/")
  
  
  resp <- fetch_url |> 
    request() |>                                   # Set up request the url
    req_error(is_error = function(resp) FALSE) |>  # Error control
    req_perform()                                  # Perform the request
  
  
  raw_data <- resp |> 
    resp_body_json(check_type = FALSE, encoding = "utf-8") # Convert json to R-list
  
  class(raw_data) # List
  
  names(raw_data) # All list element names
  
  length(raw_data$partier) # Number of parties
  
  names(raw_data$partier[[1]]) # Variables of each party
  
  party_df <- list()
  
  for(i in 1:length(raw_data$partier)) {
    
    party_df[[i]] <- data.frame(
      party_category = raw_data$partier[[i]]$id$partikategori,
      party_code     = raw_data$partier[[i]]$id$partikode,
      party_name     = raw_data$partier[[i]]$id$navn,
      percent_votes  = raw_data$partier[[i]]$stemmer$resultat$prosent,
      percent_change = raw_data$partier[[i]]$stemmer$resultat$endring$samme,
      total_votes    = raw_data$partier[[i]]$stemmer$resultat$antall$total,
      early_votes    = raw_data$partier[[i]]$stemmer$resultat$antall$fhs
    ) # Also add the other variables here (e.g raw_data$partier[[i]]$mandater$resultat$antall)
    
  }
  
  party_df <- do.call(rbind, party_df)
  
  return(party_df)
}

get_election_result(year = "2013", election = "st")
```
