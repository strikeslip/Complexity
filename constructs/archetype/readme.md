SOS - MiniSeed Data Retrieval URL Changes .iris to .earthscope 

// OLD: 
const url = `https://service.iris.edu/fdsnws/dataselect/1/query?net=${net}&sta=${sta}&loc=${loc}&cha=${cha}&starttime=${start}&endtime=${end}&format=miniseed`;

// NEW:
const url = `https://service.earthscope.org/fdsnws/dataselect/1/query?net=${net}&sta=${sta}&loc=${loc}&cha=${cha}&starttime=${start}&endtime=${end}&format=miniseed`;
