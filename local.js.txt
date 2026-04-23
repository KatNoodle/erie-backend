// This tells Vercel to cache this data globally in its memory
let globalCache = null;
let lastFetchTime = 0;
const CACHE_HOURS = 24;
// THE UNIQUE TSV LINK FOR LOCAL EXPERIENCES GOES HERE:
const SHEET_URL = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vS4PxMMqBMapxichAK0yVLbhfB6vtQPeWMrsBL_TDS881oVZ_DSNVVXx8-zT9ME0Kh8Pvx5na2HKy2I/pub?gid=0&single=true&output=tsv';

export default async function handler(req, res) {
    // 1. SECURITY (CORS): This tells Vercel to only talk to your Arrow & East website
    res.setHeader('Access-Control-Allow-Credentials', true);
    res.setHeader('Access-Control-Allow-Origin', 'https://www.erie-pa-centeroftheuniverse.com');
    res.setHeader('Access-Control-Allow-Methods', 'GET,OPTIONS');

    // Handle standard browser pre-flight checks
    if (req.method === 'OPTIONS') {
        res.status(200).end();
        return;
    }

    const now = Date.now();
    const cacheValidTime = CACHE_HOURS * 60 * 60 * 1000;

    // 2. CHECK CACHE: Does Vercel already have fresh data saved?
    if (globalCache && (now - lastFetchTime < cacheValidTime)) {
        return res.status(200).send(globalCache);
    }

    // 3. FETCH: If the cache is old, grab fresh data from Google Sheets
    try {
        const fetchResponse = await fetch(SHEET_URL);
        const data = await fetchResponse.text();
        
        // Save the new data to Vercel's global memory for the next 24 hours
        globalCache = data;
        lastFetchTime = now;
        
        // Send it to your visitor
        res.status(200).send(globalCache);
    } catch (error) {
        console.error("Failed to fetch from Google", error);
        res.status(500).send("Error loading directory data.");
    }
}