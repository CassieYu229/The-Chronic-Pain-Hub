# The Chronic Pain Hub - Complete Architecture Documentation

**Generated**: April 29, 2026  
**Version**: v0.2.0 (BioLORD-2023-M)

---

## 📁 Project File Structure

```
The-Chronic-Pain-Hub-1/
├── Frontend/                      # Frontend files (HTML + JS + CSS)
│   ├── index.html                 # Main page
│   ├── module1.html               # Module 1: Pain description analysis
│   ├── module2.html               # Module 2: Pain mapping (intro page)
│   ├── module2-interactive.html   # Module 2: Interactive pain mapping
│   ├── module3.html               # Module 3: Voice depression detection
│   ├── module4.html               # Module 4: Healthcare provider search
│   ├── css/
│   │   └── styles.css            # Global styles
│   ├── js/
│   │   ├── config.js             # API configuration
│   │   ├── script.js             # Common utilities
│   │   ├── module2-interactive.js # Module 2 logic
│   │   ├── module3.js            # Module 3 logic
│   │   ├── module4.js            # Module 4 logic
│   │   └── module4-data.js       # Module 4 static data
│   └── images/                    # Image assets
│       ├── universityofwisconsinmadisonlogo.png
│       ├── Lab_logo.png
│       ├── body-front.png        # Body front view
│       └── body-back.png         # Body back view
│
├── Backend/                       # Backend services (FastAPI)
│   ├── main.py                   # FastAPI main entry point (9 endpoints)
│   ├── .env                      # Environment variables (API keys)
│   │
│   ├── services/                 # Business logic service layer
│   │   ├── whisper_service.py           # Whisper speech-to-text
│   │   ├── llm_service.py               # OpenAI GPT service
│   │   ├── neuro_symbolic_service.py    # Neuro-symbolic reasoning engine
│   │   ├── depression_detection_service.py # Depression detection (wav2vec2)
│   │   ├── pain_mapping_service.py      # Pain mapping analysis
│   │   ├── semantic_distance_service_biolord.py  # BioLORD medical embeddings
│   │   └── semantic_distance_service_v2.py       # OpenAI embeddings (backup)
│   │
│   ├── models/                   # Pydantic data models
│   │   ├── pain_schema.py        # Pain ontology data structures
│   │   └── pain_mapping.py       # Pain mapping data structures
│   │
│   ├── ontology/                 # Pain terminology ontology
│   │   ├── pain_mapping.py              # Chinese term mapping
│   │   ├── pain_mapping_multilingual.py # Multilingual term mapping
│   │   └── mcgill_translations.py       # McGill Pain Questionnaire translations
│   │
│   ├── pipeline/                 # Analysis pipeline
│   │   └── pain_assessment_pipeline.py  # Pain assessment pipeline
│   │
│   ├── inference/                # Inference engine
│   │   └── rule_engine.py        # Clinical rule engine
│   │
│   ├── utils/                    # Utility functions
│   │   ├── language_detector.py  # Language detection
│   │   └── report_generator.py   # Report generation
│   │
│   ├── scripts/                  # Data processing scripts
│   │   ├── multilingual_pain_data.json  # Multilingual pain terminology database
│   │   ├── parse_multilingual_data.py   # Term parsing tool
│   │   └── format_pain_descriptors.py   # Term formatting tool
│   │
│   └── images/                   # Backend reference images
│       ├── sharp.png             # Sharp pain illustration
│       ├── burning.gif           # Burning pain animation
│       └── ...                   # Other pain type images
│
├── models/                       # Machine learning models
│   └── classifier_v1.pkl         # Depression detection classifier (LogisticRegression)
│
├── features/                     # Training feature data
│   ├── embeddings.npy
│   ├── labels.npy
│   ├── participant_ids.npy
│   └── processed_ids.npy
│
├── voice-screening/              # Experimental and training code
│   ├── train.py                  # Model training script
│   ├── batch_test.py             # Batch testing
│   └── ...                       # Other research code
│
├── start-all-modules.bat         # Startup script (Backend + Frontend)
└── requirements.txt              # Python dependencies
```

---

## 🔧 Backend API Endpoint List

### Backend/main.py

**FastAPI Application Configuration**:
- Title: "Pain Report Platform"
- Version: "0.2.0"
- Port: 8000
- CORS: All origins allowed

**API Endpoint List**:

| Endpoint | Method | Function | Module |
|----------|--------|----------|--------|
| `/health` | GET | Health check | All |
| `/analyze` | POST | Depression detection (audio) | Module 3 |
| `/api/analyze-text-neuro-symbolic` | POST | Text pain analysis | Module 1 |
| `/api/analyze-audio-neuro-symbolic` | POST | Audio pain analysis | Module 1 |
| `/api/system-info` | GET | System information | Module 1 |
| `/api/nppes` | GET | NPPES provider search proxy | Module 4 |
| `/api/pain-mapping/save` | POST | Save pain mapping | Module 2 |
| `/api/pain-mapping/generate-report` | POST | Generate pain report | Module 2 |
| `/api/pain-mapping/pain-types` | GET | Get pain types | Module 2 |

**Middleware**:
- `add_no_cache_headers()`: Prevents static file caching

**Startup Events**:
- `startup_event()`: Precompute BioLORD embeddings (222 McGill terms)

---

## 📦 Backend File Details

### 1. Backend/services/

#### whisper_service.py
**Purpose**: Speech-to-text using OpenAI Whisper

**Functions**:
- `transcribeAudio(audioBytes, language=None)` → dict
  - Input: Audio byte stream
  - Output: {text, language, duration}
  - Supports: Auto language detection or specified language

---

#### llm_service.py
**Purpose**: OpenAI GPT service layer for pain entity extraction and text processing

**Functions**:
1. `match_pain_terms_from_vocabulary(text, vocabulary, language)` → dict
   - Match pain terms from vocabulary
   - Input: Patient text, vocabulary, language
   - Output: List of matched terms

2. `extract_pain_entities_constrained(text)` → dict
   - Extract pain entities using GPT-5.2
   - Input: English text
   - Output: {descriptors, locations, temporal, severity, interference, emotional}
   - **Dependency**: OPENAI_API_KEY

3. `normalize_transcription(text, language)` → dict
   - Normalize transcribed text (fix spelling, grammar)
   - Input: Raw text, language
   - Output: {normalized_text, corrections, confidence}

4. `translate_pain_description(text, source_language, matched_terms, mappings)` → str
   - Translate pain description to English
   - Preserve medical terminology accuracy

5. `generate_comprehensive_report(...)` → dict
   - Generate complete clinical report
   - Includes treatment suggestions, specialist recommendations, lifestyle advice

---

#### neuro_symbolic_service.py
**Purpose**: Neuro-symbolic reasoning engine coordinating entire pain analysis workflow

**Functions**:
1. `get_pipeline()` → PainAssessmentPipeline
   - Singleton pattern to get analysis pipeline

2. `analyze_pain_neuro_symbolic(patient_text)` → dict
   - Main analysis function coordinating all steps
   - Workflow:
     1. Language detection
     2. Text normalization
     3. Vocabulary matching
     4. Term translation
     5. GPT entity extraction
     6. Rule engine inference
     7. Report generation
   - Output: Complete explainable report

3. `get_system_info()` → dict
   - Return system configuration information
   - Includes: Supported languages, term counts, model versions

---

#### depression_detection_service.py
**Purpose**: Voice depression detection (using wav2vec2 model)

**Functions**:
1. `preprocess_audio(file_path, target_sample_rate=16000)` → Tensor
   - Audio preprocessing: mono conversion, resampling

2. `get_embedding_chunked(waveform)` → Tensor
   - Extract wav2vec2 embeddings
   - Chunked processing (30 seconds/chunk)
   - Output: 768-dimensional feature vector

3. `analyze_depression_from_audio(audio_path)` → dict
   - Main analysis function
   - Input: Audio file path
   - Output: {
       prediction: 0/1,
       depression_probability: float,
       confidence: float,
       interpretation: str
     }
   - **Model**: models/classifier_v1.pkl (LogisticRegression)

---

#### pain_mapping_service.py
**Purpose**: Pain mapping data analysis and report generation

**Class**: `PainMappingService`

**Methods**:
1. `calculate_statistics(pain_data)` → dict
   - Calculate pain statistics
   - Output: {
       total_strokes: int,
       sensation_breakdown: dict,
       neuropathic_indicators: dict,
       overall_intensity: float
     }

2. `generate_report(pain_data)` → PainReport
   - Generate pain mapping report
   - Includes: Natural language summary, region analysis, neuropathic pain assessment, specialist recommendations

**Constants**:
- `PAIN_TYPE_CLINICAL`: Clinical significance mapping for pain types

---

#### semantic_distance_service_biolord.py
**Purpose**: BioLORD-2023-M medical semantic embedding service (default)

**Functions**:
1. `load_biolord_model()` → SentenceTransformer
   - Load BioLORD model
   - Model: FremyCompany/BioLORD-2023-M
   - Size: ~1GB
   - Dimensions: 768

2. `precompute_dictionary_embeddings()`
   - Precompute McGill translation embeddings
   - Cache 222 terms (Chinese: 78, Korean: 47, Spanish: 53, Hmong: 44)

3. `cosine_similarity(vec1, vec2)` → float
   - Calculate cosine similarity

4. `calculate_semantic_distances(patient_terms, language)` → dict
   - Calculate semantic distances between patient terms and McGill terms
   - Helps improve mapping accuracy

5. `analyze_all_terms_with_biolord(text, language)` → list
   - Analyze semantic matching for all terms

---

#### semantic_distance_service_v2.py
**Purpose**: OpenAI embedding service (backup)

**Functions**:
1. `precompute_dictionary_embeddings()`
   - Use OpenAI text-embedding-3-large
   - Dimensions: 3072

2. `calculate_semantic_distances(patient_terms, language)` → dict
   - Same as above, using OpenAI embeddings

3. `cosine_similarity(a, b)` → float
   - Calculate cosine similarity

---

### 2. Backend/models/

#### pain_schema.py
**Purpose**: Pain ontology data structure definitions

**Pydantic Classes**:
1. `PainOntology(BaseModel)`
   - Fields: descriptors, locations, temporal_pattern, severity, interference, emotional_impact, body_region, pain_type, characteristics

2. `ClinicalRecommendation(BaseModel)`
   - Fields: specialist_type, reason, urgency_level, tests, treatments, lifestyle

3. `ExplainableReport(BaseModel)`
   - Fields: summary, pain_ontology, clinical_recommendations, reasoning_chain, confidence_scores, system_metadata

---

#### pain_mapping.py
**Purpose**: Pain mapping data structures

**Pydantic Classes**:
1. `PainStroke(BaseModel)`
   - Fields: x, y, sensation_type, intensity, timestamp

2. `PainRegion(BaseModel)`
   - Fields: x, y, width, height, body_view, label

3. `PainMapData(BaseModel)`
   - Fields: session_id, front_strokes, back_strokes, marked_regions, patient_notes, timestamp, total_strokes, sensation_breakdown, neuropathic_indicators, overall_intensity

4. `PainReport(BaseModel)`
   - Fields: summary, affected_regions, neuropathic_probability, specialist_recommendations, timestamp

---

### 3. Backend/ontology/

#### pain_mapping.py
**Purpose**: Chinese pain term mapping to English

**Functions**:
1. `map_chinese_to_english(chinese_text)` → List[Dict]
   - Map Chinese terms to English McGill terms

2. `extract_temporal_pattern(chinese_text)` → Optional[str]
   - Extract temporal patterns (continuous, intermittent, paroxysmal)

3. `extract_anatomical_location(chinese_text)` → List[str]
   - Extract anatomical locations

4. `get_unmapped_descriptors(chinese_text, mappings)` → List[str]
   - Find unmapped terms

5. `suggest_similar_terms(unmapped_term, language)` → List[Dict]
   - Suggest similar terms

6. `verify_all_terms_with_semantic_model(...)` → dict
   - Verify all terms using semantic model

**Data**: CHINESE_PAIN_DESCRIPTORS (200+ terms)

---

#### pain_mapping_multilingual.py
**Purpose**: Multilingual pain term mapping (Chinese, Korean, Spanish, Hmong)

**Functions**:
1. `_load_pain_descriptors()` → dict
   - Load from multilingual_pain_data.json

2. `_extract_core_terms(term, language)` → List[str]
   - Extract core terms (remove filler words)

3. `map_multilingual_to_english(text, source_language, use_semantic)` → dict
   - Main multilingual mapping function
   - Supports: Chinese, Korean, Spanish, Hmong
   - Output: {mappings, unmapped, language, translation_notes}

4. `get_supported_languages()` → List[str]
   - Return list of supported languages

5. `get_descriptor_count(language)` → int
   - Return term count for specific language

**Data Source**: scripts/multilingual_pain_data.json
- Chinese: 88 terms
- Korean: 48 terms
- Spanish: 54 terms
- Hmong: 45 terms

---

#### mcgill_translations.py
**Purpose**: McGill Pain Questionnaire multilingual translations

**Data Structure**: 
```python
MCGILL_TRANSLATIONS = {
    "Chinese": {
        "刺痛": "pricking",
        "锐痛": "sharp",
        ...  # 78 terms
    },
    "Korean": {
        "찌르는": "pricking",
        ...  # 47 terms
    },
    "Spanish": {
        "punzante": "pricking",
        ...  # 53 terms
    },
    "Hmong": {
        "mob nkhaus": "stabbing",
        ...  # 44 terms
    }
}
```

**Total**: 222 McGill term translations

---

### 4. Backend/pipeline/

#### pain_assessment_pipeline.py
**Purpose**: Main pain assessment pipeline coordinating all components

**Class**: `PainAssessmentPipeline`

**Main Methods**:
1. `assess(patient_text, source_language)` → ExplainableReport
   - Complete assessment workflow
   - Steps:
     1. Term mapping
     2. Text translation
     3. Entity extraction
     4. Ontology construction
     5. Rule inference
     6. Report generation

2. `_build_reasoning_chain(...)` → List[str]
   - Build reasoning chain (explainability)

3. `_calculate_confidence_scores(...)` → dict
   - Calculate confidence scores

---

### 5. Backend/inference/

#### rule_engine.py
**Purpose**: Clinical rule engine recommending specialists based on symptoms

**Class**: `ClinicalRule`
- Fields: name, condition, action, priority

**Class**: `RuleEngine`

**Methods**:
1. `add_rule(rule: ClinicalRule)`
   - Add rule

2. `evaluate(pain_ontology: PainOntology)` → ClinicalRecommendation
   - Evaluate pain ontology, return clinical recommendations

**Predefined Rules**:
- Neuropathic pain → Neurology
- Inflammatory pain → Rheumatology
- Musculoskeletal pain → Orthopedics/Physical Therapy
- Headache → Neurology
- Abdominal pain → Gastroenterology/Gynecology
- Chest pain → Cardiology (urgent)
- Chronic pain → Pain Management specialty

---

### 6. Backend/utils/

#### language_detector.py
**Purpose**: Multilingual detection

**Functions**:
1. `detect_language(text)` → LanguageCode
   - Detect text language
   - Supports: English, Chinese, Korean, Spanish, Hmong
   - Method: Character set analysis + keyword matching

2. `get_language_name(code)` → str
   - Convert language code to language name

---

#### report_generator.py
**Purpose**: Clinical report generation

**Functions**:
1. `generate_comprehensive_report(pain_ontology, mappings, ...)` → dict
   - Generate detailed clinical report
   - Includes: Natural language summary, specialist recommendations, treatment suggestions, lifestyle adjustments

2. `translate_to_english_simple(text)` → str
   - Simple translation (backup)

---

### 7. Backend/scripts/

#### multilingual_pain_data.json
**Purpose**: Multilingual pain terminology database

**Structure**:
```json
{
  "languages": {
    "Chinese": {
      "name": "中文",
      "descriptors": [
        {
          "foreign": "刺痛",
          "english": "sharp pain",
          "pain_type": "neuropathic",
          "mcgill_category": "sensory",
          "clinical_significance": "..."
        },
        ...
      ]
    },
    "Korean": {...},
    "Spanish": {...},
    "Hmong": {...}
  }
}
```

#### parse_multilingual_data.py
**Purpose**: Parse terms from Excel to JSON

**Functions**:
1. `categorize_pain_type(english_word)` → str
   - Categorize pain type based on English word

2. `parse_sheet(xlsx_path, sheet_name, english_col, foreign_col)` → list
   - Parse Excel sheet

3. `main()`
   - Main processing workflow

---

## 🎨 Frontend File Details

### Frontend/js/

#### config.js
**Purpose**: API configuration

**Exports**:
- `window.API_CONFIG.BASE_URL`: API base URL (auto-detect localhost or production)
- `getApiBase()`: Get API base URL function

---

#### script.js
**Purpose**: Common utility functions (shared across main page and modules)

**Functions**:
1. `escapeHtml(value)` → str
   - HTML escaping (prevent XSS)

2. `getProviderName(item)` → str
   - Get provider name

3. `getPreferredAddress(addresses)` → object
   - Get preferred address (mailing address priority)

4. `formatProviderAddress(address)` → str
   - Format address string

5. `getProviderSpecialty(item)` → str
   - Get specialty

6. `normalizeSupportProvider(item)` → object
   - Normalize NPPES data

7. `fetchSupportProviders(zip, need)` → Promise<array>
   - Fetch provider list from NPPES API

8. `createSupportResultCard(provider)` → HTMLElement
   - Create provider card HTML

9. `renderSupportResults()`
   - Render search results (paginated)

10. `runSupportSearch()`
    - Execute search (main page search box)

**Event Listeners**:
- Smooth scrolling (anchor links)
- IntersectionObserver (scroll animations)

---

#### module2-interactive.js
**Purpose**: Module 2 interactive pain mapping

**Constants**:
- `API_BASE`: API base URL
- `PAIN_TYPES`: Pain type configuration (colors, labels, clinical significance)

**Global State**:
```javascript
state = {
  selectedTool: 'sharp',
  currentIntensity: 5,
  isDrawing: false,
  frontStrokes: [],
  backStrokes: [],
  markedRegions: [],
  sessionId: generateSessionId()
}
```

**Functions**:
1. `initializeCanvases()`
   - Initialize front/back view canvases

2. `loadPainTools()`
   - Load pain type toolbar

3. `loadColorLegend()`
   - Load color legend

4. `selectTool(tool)`
   - Select pain type

5. `setupEventListeners()`
   - Setup mouse/touch events

6. `startDrawing(e, view)`, `draw(e, view)`, `stopDrawing()`
   - Drawing logic

7. `updateAnalysis()`
   - Real-time statistics update

8. `updateIntensity(value)`
   - Update intensity slider

9. `clearCanvas()`
   - Clear canvas

10. `saveProgress()`
    - Save progress (call `/api/pain-mapping/save`)

11. `generateReport()`
    - Generate report (call `/api/pain-mapping/generate-report`)

12. `collectPainData()` → PainMapData
    - Collect all drawing data

13. `displayReport(report)`
    - Display report modal

14. `closeModal()`, `downloadReport()`
    - Modal operations

15. `fetchPainTypesFromAPI()`
    - Fetch pain types from API

16. `generateSessionId()` → str
    - Generate session ID

---

#### module3.js
**Purpose**: Module 3 voice depression detection

**Constants**:
- `API`: Backend API URL

**Global State**:
```javascript
{
  currentFile: null,
  isRecording: false,
  mediaRecorder: null,
  recordingChunks: [],
  recordingStartTime: null
}
```

**Functions**:
1. `handleFile(file)`
   - Handle file selection/drop

2. `showResults(data)`
   - Display analysis results
   - Render: prediction, probabilities, confidence, interpretation

**Event Listeners**:
- File selection
- Drag and drop upload
- Recording (using MediaRecorder API)
- Analyze button (call `/analyze`)
- Reset

**Recording Feature**:
- Use `navigator.mediaDevices.getUserMedia()`
- Recording format: audio/webm
- Auto timer

---

#### module4.js
**Purpose**: Module 4 healthcare provider search

**Constants**:
- `MODULE4_BACKEND_BASE_URL`: Backend URL
- `NPI_PAIN_QUERY`: NPPES query parameters (pain management specialty)

**Functions**:
1. `formatDisplayDate(dateString)` → str
   - Format date

2. `buildBackendNpiUrl()` → str
   - Build NPPES API URL

3. `prettifyLabel(value)` → str
   - Prettify label display

4. `getPreferredAddress(addresses)` → object
   - Get preferred address

5. `getProviderName(item)` → str
   - Get provider name

6. `getProviderSpecialty(item)` → str
   - Get specialty

7. `hasMedicaidIdentifier(item)` → bool
   - Check if accepts Medicaid

8. `normalizeNppesProvider(item)` → object
   - Normalize NPPES data

9. `fetchLivePainProviders()` → Promise<object>
   - Fetch pain management providers (call `/api/nppes`)

10. `createMetricCard(label, value)` → HTMLElement
    - Create metric card

11. `createPriorityItem(item)` → HTMLElement
    - Create priority area card

12. `createShortageItem(item)` → HTMLElement
    - Create shortage area card

13. `createProviderCard(provider)` → HTMLElement
    - Create provider card

14. `renderProviderGrid(npi)` → void
    - Render provider grid

15. `renderStaticSnapshot(data, npi, generatedDate, isLiveBackend)` → void
    - Render static snapshot

16. `populateModule4()` → Promise<void>
    - Populate data on page load

**Page Load**: `DOMContentLoaded` → `populateModule4()`

---

#### module4-data.js
**Purpose**: Module 4 static data (Wisconsin healthcare shortage areas)

**Data Structure**:
```javascript
{
  generated_date: "2025-01-21",
  wisconsin_snapshot: {
    total_hpsa_areas: 127,
    affected_counties: 40,
    population_affected: "~1.2M",
    shortage_areas: [
      {
        name: "Northwoods Primary Care HPSA",
        type: "Primary Care",
        ...
      }
    ],
    priority_counties: [...]
  }
}
```

---

## 🔄 Module Call Chain Details

### Module 1: Pain Description Analysis (Text/Audio)

#### Text Analysis Workflow

```
User Input Text
    ↓ (Frontend/module1.html inline script)
Click "Analyze"
    ↓
API Call: POST /api/analyze-text-neuro-symbolic
    ↓
Backend/main.py → analyzeTextNeuroSymbolic()
    ↓
Backend/services/neuro_symbolic_service.py → analyze_pain_neuro_symbolic(text)
    ├─→ 1. Backend/utils/language_detector.py → detect_language(text)
    │      Returns: LanguageCode (Chinese/English/Korean/Spanish/Hmong)
    │
    ├─→ 2. Backend/services/llm_service.py → normalize_transcription(text, language)
    │      Calls: OpenAI GPT-4 API
    │      Returns: {normalized_text, corrections, confidence}
    │
    ├─→ 3. Backend/ontology/pain_mapping_multilingual.py → map_multilingual_to_english(text, language)
    │      ├─→ Loads: Backend/scripts/multilingual_pain_data.json
    │      ├─→ Fuzzy matching (fuzzywuzzy)
    │      └─→ Semantic matching: Backend/services/semantic_distance_service_biolord.py
    │            └─→ BioLORD-2023-M model calculates similarity
    │      Returns: {mappings: [{foreign, english, confidence, pain_type}], unmapped}
    │
    ├─→ 4. Backend/services/llm_service.py → translate_pain_description(text, language, matched_terms, mappings)
    │      Calls: OpenAI GPT-4 API
    │      Returns: English translation
    │
    ├─→ 5. Backend/services/llm_service.py → extract_pain_entities_constrained(english_text)
    │      Calls: OpenAI GPT-5.2 API (JSON mode)
    │      Returns: {descriptors, locations, temporal, severity, interference, emotional}
    │
    ├─→ 6. Backend/pipeline/pain_assessment_pipeline.py → assess(text, language)
    │      ├─→ Builds: PainOntology object (Backend/models/pain_schema.py)
    │      ├─→ Calls: Backend/inference/rule_engine.py → evaluate(pain_ontology)
    │      │      Matches clinical rules → Returns ClinicalRecommendation
    │      └─→ Generates: ExplainableReport
    │
    └─→ 7. Backend/utils/report_generator.py → generate_comprehensive_report(...)
           Generates: Natural language report + specialist recommendations + treatment suggestions
    ↓
Return to Frontend: {
    status: "success",
    transcription: {...},
    structured_data: PainOntology,
    clinical_recommendations: [...],
    reasoning_chain: [...],
    confidence_scores: {...}
}
    ↓
Frontend: Render results (Frontend/module1.html inline script)
```

#### Audio Analysis Workflow

```
User Upload Audio
    ↓ (Frontend/module1.html)
Click "Analyze Audio"
    ↓ (Frontend/module1.html inline script)
API Call: POST /api/analyze-audio-neuro-symbolic (FormData)
    ↓
Backend/main.py → analyzeAudioNeuroSymbolic(file)
    ├─→ 1. Backend/services/whisper_service.py → transcribeAudio(audioBytes)
    │      Calls: OpenAI Whisper API
    │      Returns: {text: "...", language: "zh", duration: 10.5}
    │
    └─→ 2. Calls text analysis workflow (same as above)
           Backend/services/neuro_symbolic_service.py → analyze_pain_neuro_symbolic(text)
    ↓
Return to Frontend: {
    status: "success",
    whisper_language: "zh",
    filename: "audio.wav",
    ... (same as text analysis results)
}
    ↓
Frontend: Render results
```

---

### Module 2: Interactive Pain Mapping

```
User Opens Page
    ↓ (Frontend/module2-interactive.html)
Frontend: DOMContentLoaded (Frontend/js/module2-interactive.js)
    ├─→ initializeCanvases() - Create front/back view canvases
    ├─→ loadPainTools() - Load pain type toolbar
    ├─→ loadColorLegend() - Load color legend
    ├─→ fetchPainTypesFromAPI() - Get pain type definitions
    │      API Call: GET /api/pain-mapping/pain-types
    │      Backend/main.py → getPainTypes()
    │      Backend/services/pain_mapping_service.py → PAIN_TYPE_CLINICAL
    │      Returns: {sharp, burning, tingling, throbbing, ...}
    └─→ setupEventListeners() - Setup drawing events
    ↓
User Draws Pain Areas
    ├─→ Select tool: selectTool('sharp')
    ├─→ Mouse drag: startDrawing() → draw() → stopDrawing()
    ├─→ Record strokes: state.frontStrokes.push({x, y, sensation_type, intensity, timestamp})
    └─→ Real-time analysis: updateAnalysis() - Calculate statistics
    ↓
User Clicks "Save Progress"
    ↓
Frontend/js/module2-interactive.js: saveProgress()
    ├─→ collectPainData() - Collect all data
    │      Returns: PainMapData (Backend/models/pain_mapping.py) {session_id, front_strokes, back_strokes, marked_regions, ...}
    ↓
API Call: POST /api/pain-mapping/save
    ↓
Backend/main.py → savePainMapping(pain_data)
    ├─→ Backend/services/pain_mapping_service.py → calculate_statistics(pain_data)
    │      Calculates: total_strokes, sensation_breakdown, neuropathic_indicators, overall_intensity
    │      Returns: {total_strokes: 45, sensation_breakdown: {sharp: 20, burning: 15, ...}, ...}
    └─→ Update pain_data object (can be saved to database in future)
    ↓
Return to Frontend: {status: "success", statistics: {...}}
    ↓
User Clicks "Generate Report"
    ↓
Frontend/js/module2-interactive.js: generateReport()
    ├─→ collectPainData() - Collect complete data
    ↓
API Call: POST /api/pain-mapping/generate-report
    ↓
Backend/main.py → generatePainReport(pain_data)
    ↓
Backend/services/pain_mapping_service.py → generate_report(pain_data)
    ├─→ Analyze front/back view data
    ├─→ Identify affected regions (neck, back, limbs, etc.)
    ├─→ Calculate neuropathic pain probability (tingling + shooting ratio)
    ├─→ Generate natural language summary
    └─→ Recommend specialist physicians
    ↓
Return to Frontend: PainReport (Backend/models/pain_mapping.py) {
    summary: "Patient reported 45 pain marks...",
    affected_regions: ["upper_back", "neck", ...],
    neuropathic_probability: 0.35,
    specialist_recommendations: ["Neurology", "Pain Management"],
    timestamp: "2026-04-29T..."
}
    ↓
Frontend/js/module2-interactive.js: displayReport(report) - Show modal
    ↓
User can: closeModal() / downloadReport() (generate PDF)
```

---

### Module 3: Voice Depression Detection

```
User Opens Page
    ↓ (Frontend/module3.html)
Frontend: DOMContentLoaded (Frontend/js/module3.js)
    └─→ Setup event listeners (file upload, recording, analysis)
    ↓
User Uploads Audio File OR Records Audio
    ├─→ File upload: handleFile(file)
    │      Sets: state.currentFile = file
    │      Shows: filename + "Analyze" button
    │
    └─→ Recording feature:
           ├─→ Click "Start Recording"
           ├─→ navigator.mediaDevices.getUserMedia({audio: true})
           ├─→ MediaRecorder.start() - Start recording
           ├─→ Auto timer display
           ├─→ Click "Stop Recording"
           ├─→ MediaRecorder.stop() - Stop recording
           └─→ Generate Blob → Create File object → handleFile(file)
    ↓
User Clicks "Analyze Audio"
    ↓
Frontend/js/module3.js: analyze button event
    ├─→ Create FormData: formData.append('file', state.currentFile)
    ↓
API Call: POST /analyze (multipart/form-data)
    ↓
Backend/main.py → analyze_depression(file)
    ├─→ Validate: Check file type (audio/*)
    ├─→ Save temp file: tempfile.NamedTemporaryFile(suffix='.wav')
    ├─→ Format conversion: pydub.AudioSegment → WAV format
    │      (Handle webm/ogg browser recording formats)
    ↓
Backend/services/depression_detection_service.py → analyze_depression_from_audio(audio_path)
    ├─→ 1. preprocess_audio(audio_path, 16000)
    │      ├─→ torchaudio.load() - Load audio
    │      ├─→ Mono conversion: waveform.mean(dim=0)
    │      └─→ Resample to 16kHz: torchaudio.transforms.Resample()
    │
    ├─→ 2. get_embedding_chunked(waveform)
    │      ├─→ Chunk: 30 seconds per chunk (CHUNK_SIZE = 30 * 16000)
    │      ├─→ For each chunk:
    │      │   ├─→ Wav2Vec2Processor.process() - Preprocess
    │      │   ├─→ Wav2Vec2Model.forward() - Extract embeddings
    │      │   └─→ outputs.last_hidden_state.mean(dim=1) - Mean pooling
    │      └─→ torch.stack().mean() - Average embeddings of all chunks (768-dim)
    │
    └─→ 3. classifier_pipeline.predict(embedding)
           ├─→ Load: models/classifier_v1.pkl
           ├─→ Pipeline: [StandardScaler, PCA, LogisticRegression]
           ├─→ prediction = model.predict(embedding)[0]  # 0 or 1
           ├─→ probabilities = model.predict_proba(embedding)[0]  # [prob_no_depression, prob_depression]
           └─→ Returns: {
                  prediction: 0/1,
                  depression_probability: probabilities[1],
                  not_depression_probability: probabilities[0],
                  confidence: max(probabilities),
                  interpretation: "..."
               }
    ↓
Backend/main.py: Cleanup temp file (os.unlink)
    ↓
Return to Frontend: {
    prediction: 0,  # 0=No Depression, 1=Depression
    depression_probability: 0.23,
    not_depression_probability: 0.77,
    confidence: 0.77,
    interpretation: "Low depression risk (23%)"
}
    ↓
Frontend/js/module3.js: showResults(data)
    ├─→ Display prediction: "No depression detected" / "Depression indicators detected"
    ├─→ Display probability bar: Visualize depression probability
    ├─→ Display confidence: "Confidence: 77%"
    └─→ Display interpretation: "Low depression risk..."
```

---

### Module 4: Healthcare Provider Search

```
User Opens Page
    ↓ (Frontend/module4.html)
Frontend: DOMContentLoaded (Frontend/js/module4.js)
    └─→ populateModule4()
    ↓
Frontend/js/module4.js: fetchLivePainProviders()
    ├─→ buildBackendNpiUrl()
    │      Builds query: /api/nppes?version=2.1&postal_code=53711&limit=25&taxonomy_description=Pain*
    ↓
API Call: GET /api/nppes?version=2.1&postal_code=53711&limit=25&taxonomy_description=Pain*
    ↓
Backend/main.py → proxyNPPES(request)
    ├─→ Extract query params: dict(request.query_params)
    ├─→ Build NPPES URL: https://npiregistry.cms.hhs.gov/api/?...
    ├─→ httpx.AsyncClient.get(nppes_url)  # Async HTTP request
    └─→ Returns: Response(content=nppes_response, status_code=200)
    ↓
External API: NPPES Registry (CMS API)
    └─→ Query National Provider Identifier database
    └─→ Returns: {
           result_count: 15,
           results: [
             {
               number: "1234567890",
               basic: {name: "Dr. John Smith", ...},
               addresses: [...],
               taxonomies: [{desc: "Pain Management"}],
               ...
             },
             ...
           ]
        }
    ↓
Return to Frontend: NPPES JSON data
    ↓
Frontend/js/module4.js: fetchLivePainProviders() continues processing
    ├─→ Normalize data: results.map(normalizeNppesProvider)
    │      Extracts: name, specialty, address, phone, npi, medicaid_accepted
    ├─→ Calculate statistics: {
    │      total_pain_specialists: 15,
    │      medicaid_accepting: 8,
    │      avg_distance: "~12 miles"
    │   }
    └─→ Returns: {npi: {...}, wisconsin_snapshot: {...}, generated_date: "2026-04-29"}
    ↓
Frontend/js/module4.js: renderProviderGrid(npi)
    ├─→ For each provider: createProviderCard(provider)
    │      Generates HTML card: name, specialty, address, phone, NPI, Medicaid badge
    └─→ Insert to page: #provider-grid
    ↓
Frontend/js/module4.js: renderStaticSnapshot(data)
    ├─→ Load Wisconsin shortage data from Frontend/js/module4-data.js
    ├─→ Render metric cards: total_hpsa_areas, affected_counties, population_affected
    ├─→ Render priority county list: priority_counties
    └─→ Render shortage area list: shortage_areas
    ↓
Page displays complete provider list + Wisconsin healthcare shortage info
```

---

## 🔌 External Dependencies and APIs

### Required API Keys (Backend/.env)

```env
# OpenAI APIs (Module 1)
OPENAI_API_KEY=sk-proj-...              # GPT-4/GPT-5.2 + Whisper

# Optional Configuration
EMBEDDING_MODEL=biolord                  # Options: "biolord" (default) or "openai"
```

### External Service Dependencies

| Service | Purpose | Module | Status |
|---------|---------|--------|--------|
| OpenAI GPT-4/5.2 | Entity extraction, translation | Module 1 | Required |
| OpenAI Whisper | Speech-to-text | Module 1 | Required |
| OpenAI Embeddings | Semantic matching (backup) | Module 1 | Optional |
| HuggingFace Hub | BioLORD model download | Module 1 | Required |
| CMS NPPES API | Provider search | Module 4 | Public API |

### Python Dependencies

**Core Framework**:
- `fastapi` - Web framework
- `uvicorn` - ASGI server
- `pydantic` - Data validation

**AI/ML**:
- `torch` - PyTorch deep learning
- `transformers` - HuggingFace Transformers
- `sentence-transformers` - BioLORD model
- `openai` - OpenAI API client
- `whisper` - OpenAI Whisper (speech recognition)

**Audio Processing**:
- `torchaudio` - Audio loading and processing
- `pydub` - Audio format conversion
- `ffmpeg` - Audio codec (system dependency)

**Data Processing**:
- `numpy` - Numerical computing
- `pandas` - Data analysis (optional)
- `scikit-learn` - Machine learning (PCA, LogisticRegression)
- `fuzzywuzzy` - Fuzzy string matching

**HTTP**:
- `httpx` - Async HTTP client (NPPES proxy)

---

## 🚀 Startup and Deployment

### Local Development Startup

**Method 1: Using Startup Script (Recommended)**
```batch
start-all-modules.bat
```
- Backend: http://localhost:8000
- Frontend: http://localhost:3000

**Method 2: Manual Startup**
```bash
# Terminal 1: Backend
cd Backend
python main.py

# Terminal 2: Frontend
cd Frontend
python -m http.server 3000
```

### Environment Configuration

1. **Install Python Dependencies**:
```bash
pip install -r requirements.txt
```

2. **Configure Environment Variables** (Backend/.env):
```env
OPENAI_API_KEY=sk-proj-your-key-here
EMBEDDING_MODEL=biolord
```

3. **Install FFmpeg** (audio processing):
```bash
# Windows (Anaconda)
conda install ffmpeg

# Or download from https://ffmpeg.org/download.html
```

4. **First Startup**:
- BioLORD model (~1GB) will auto-download to `~/.cache/huggingface/`
- McGill term embeddings will auto-precompute (~10 seconds)


## 🔑 Key Technical Points

### 1. Neuro-Symbolic Reasoning

**Principle**: Combines neural networks (GPT) language understanding with symbolic systems (rule engine) logical reasoning

**Implementation**:
- **Neural Part**: OpenAI GPT extracts entities and relationships
- **Symbolic Part**: Rule Engine infers based on clinical rules
- **Ontology**: PainOntology as intermediate representation

**Advantages**:
- Strong explainability (reasoning_chain)
- High accuracy (combines human medical knowledge)
- Good maintainability (rules adjustable)

---

### 2. Multilingual Medical Term Mapping

**Challenge**: Cultural differences and translation ambiguity in medical terminology

**Solutions**:
1. **Static Dictionary**: multilingual_pain_data.json (235+ terms)
2. **Fuzzy Matching**: fuzzywuzzy (handles spelling variants)
3. **Semantic Matching**: BioLORD-2023-M (handles synonyms)
4. **McGill Translations**: Standardized pain descriptors

**Supported Languages**: Chinese, Korean, Spanish, Hmong, English

---

### 3. BioLORD Medical Semantic Embeddings

**Model**: FremyCompany/BioLORD-2023-M
- Based on XLM-RoBERTa
- Pre-trained on biomedical literature
- 768-dimensional vector space
- Supports multilingual medical term similarity calculation

**Uses**:
- Assists term mapping (when dictionary matching fails)
- Improves mapping accuracy
- Discovers synonyms and related terms

---

### 4. Depression Detection Audio Feature Extraction

**Model Pipeline**:
```
Raw Audio (wav/webm/ogg)
    ↓ pydub (format conversion)
WAV 16kHz Mono
    ↓ torchaudio (loading)
Waveform Tensor
    ↓ Wav2Vec2Processor (preprocessing)
Input Features
    ↓ Wav2Vec2Model (encoding)
Hidden States (768-dim)
    ↓ Mean Pooling (time dimension average)
Audio Embedding (768-dim)
    ↓ StandardScaler + PCA + LogisticRegression
Depression Prediction (0/1 + probability)
```

**Training Data**: DAIC-WOZ (Depression Corpus)

---

### 5. Real-time Pain Mapping Analysis

**Canvas Drawing Technology**:
- HTML5 Canvas API
- Mouse/touch event handling
- Dual-view sync (front/back)

**Data Structure**:
```javascript
PainStroke {
  x: number,
  y: number,
  sensation_type: 'sharp' | 'burning' | ...,
  intensity: 1-10,
  timestamp: ISO string
}
```

**Real-time Analysis**:
- Stroke counting
- Sensation type distribution
- Neuropathic indicators (tingling + shooting ratio)
- Overall intensity weighted average

---

## 🐛 Common Issues and Solutions

### 1. OpenAI API 401 Error
**Issue**: `Error code: 401 - invalid_api_key`
**Cause**: OPENAI_API_KEY expired or invalid
**Solution**: 
- Update `OPENAI_API_KEY` in `Backend/.env`
- Get new key: https://platform.openai.com/account/api-keys
- Restart Backend server

### 2. BioLORD Model Download Slow
**Issue**: First startup takes very long
**Cause**: Model ~1GB, download speed depends on network
**Solution**:
- Use HF_TOKEN to accelerate: `export HF_TOKEN=your_token`
- Or use mirror: `export HF_ENDPOINT=https://hf-mirror.com`

### 3. Module 3 Recording Not Working
**Issue**: Click "Start Recording" no response
**Cause**: 
- Browser hasn't granted microphone permission
- Only supports HTTPS or localhost
**Solution**:
- Check browser permission settings
- Use localhost or configure HTTPS

### 4. Images Not Loading (404)
**Issue**: Logo or body images not displaying
**Cause**: Path error or StaticFiles configuration issue
**Solution**: 
- Check if `Frontend/images/` files exist
- Confirm Backend `main.py` StaticFiles mount is correct
- Clear browser cache (Ctrl+Shift+R)

### 5. NPPES API Timeout
**Issue**: Module 4 provider search timeout
**Cause**: CMS NPPES API slow response
**Solution**:
- Increase timeout: `httpx.AsyncClient(timeout=20.0)`
- Add retry mechanism
- Use static data fallback (module4-data.js)

---

## 📈 Performance Optimization Suggestions

### Backend Optimization
1. **Precompute Embeddings**: Precompute McGill terms at startup (implemented)
2. **Cache LLM Responses**: Cache GPT results for common queries
3. **Async Processing**: Use `asyncio.to_thread()` to avoid blocking (implemented)
4. **Model Quantization**: Use quantized Wav2Vec2 to reduce memory

### Frontend Optimization
1. **Canvas Optimization**: Use `requestAnimationFrame` for smooth drawing
2. **Lazy Loading**: Lazy load images and data
3. **Debounce/Throttle**: Use debounce for search input
4. **Cache API Results**: Use localStorage to cache NPPES data

### Deployment Optimization
1. **CDN**: Use CDN for static resources
2. **Gzip Compression**: Enable HTTP compression
3. **Load Balancing**: Use multiple Backend instances
4. **Database**: Add database to store historical records

---

## 📝 Maintenance and Extension Guide

### Adding New Language Support
1. Add new language terms in `Backend/scripts/multilingual_pain_data.json`
2. Add McGill translations in `Backend/ontology/mcgill_translations.py`
3. Update `Backend/utils/language_detector.py` to add detection logic
4. Re-precompute BioLORD embeddings (automatic)

### Adding New Clinical Rules
1. Add new rule in `Backend/inference/rule_engine.py`:
```python
rule = ClinicalRule(
    name="new_rule",
    condition=lambda ont: ont.pain_type == "inflammatory",
    action=lambda: ClinicalRecommendation(...),
    priority=5
)
engine.add_rule(rule)
```

### Adding New Pain Types
1. Add to `PAIN_TYPES` in `Frontend/js/module2-interactive.js`
2. Add to `PAIN_TYPE_CLINICAL` in `Backend/services/pain_mapping_service.py`
3. Add corresponding illustration in `Backend/images/` (optional)

### Adding New API Endpoints
1. Add route in `Backend/main.py`:
```python
@app.post("/api/new-endpoint")
async def new_endpoint(data: RequestModel):
    # Processing logic
    return {"result": "..."}
```
2. Update frontend call code
3. Update API endpoint list in this documentation

---

## 📄 License and Acknowledgments

**Project**: The Chronic Pain Hub  
**Institution**: University of Wisconsin-Madison  
**License**: See LICENSE file

**Open Source Projects Used**:
- FastAPI - Web framework
- HuggingFace Transformers - NLP models
- BioLORD - Medical semantic embeddings
- OpenAI Whisper - Speech recognition
- CMS NPPES - Healthcare provider data

---