# SoK (Systematization of Knowledge): Prudent Evaluation Practices for Fuzzing

## Paper Content

- Analyzed 150 fuzzing papers presented at top conferences between 2018 and 2023
- Examined adherence to guidelines concerning evaluation of fuzzing strategies
-- systematically analyze how evaulations are conducted in terms of metrics, targets, baselines, reported bugs,...
-- check whether common fuzzing guidelines outlined by Klees et al. [88], ... are followed ("G. Klees, A. Ruef, B. Cooper, S. Wei, and M. Hicks, “Evaluating 
Fuzz Testing,” in ACM Conference on Computer and Communica-
tions Security (CCS), 2018.)

-- investigate potential flaws threatening validity of the evaluation
- Attempt to reproduce eight papers
-- discuss shortcomings
- Proposal of updated guidelines for evaluating fuzzing strategies

## Fuzzing, Guidelines on Fuzz Test Evaluation

- Fuzzing: Dynamic testing technique to uncover bugs
- **Randomly** derive test input by mutating previous inputs or from input specs (e.g. grammars)
- Use "oracles" to monitor "unusual" behavior in tested programs
- Observe coverage, mutate tests to maximize coverage

### Guidelines
- Recommendation 1: Compare with Baseline
- Recommendation 2: Select relevant targets (i.e. programs to test)
- Recommendation 3: Setup and Parameters - Repeat tests multiple times, run each test for a longer time, e.g. 24 hours, carefully select seed sets
- Recommendation 4: Evaluation metrics; number of found bugs, coverage as secondary metric
- Recommendation 5: Statistical Evaluation; generate statistically relevant number of trials
(
- Recommendation of FuzzBench paper: Performance of fuzzers vary significantly depending on the initial seed(s), using saturated corpus is not recommended
- Recommendaton of Böhme et al.: 10 target programs, test at least 10 times for at least 12hrs. Use real-world programs, evaluate findings (real bugs found?), split benchmarks into "training" and "validation", discuss threats to the validity & how to mitigate them, document test setups carefully and publish evaluation artifacts
M. Böhme, L. Szekeres, and J. Metzman, “On the Reliability of 
Coverage-Based Fuzzer Benchmarking,” in IEEE/ACM International 
Conference on Automated Software Engineering (ASE), 2022.
)

## Literature Analysis

- Only examine papers which focus was on fuzzing -> 289 papers
- Randomly select 150
- Verify if recommendations were followed, check if there are good reasons for not following them
- Check for flaws in the evaluation to get recommendations on how to avoid them

## Results

### Reproducibility

- 74% published code of presented technique
- 11% share obtained data
- conferences offer artifact evaluation process (independent reviewers asses research artifact); 23% of papers had their findings evaluated

### Targets under Test

- Strong bias towards byte-oriented (binary) file formats.
- On average, ~9 target programs were tested
- 753 different targets across all papers were tested, 576 of them only in on paper
- 91/61% of the papers use no benchmark, the others used LAVA-M, FuzzBench, Fuzzer TestSuite, CGC binaries, Magma, Unibench

### Evaluation against State of the Art

- Only a few techniques published in the past were incorporated
- Mostly, existing fuzzers have been extended: AFL(++), libFuzzer, syzkaller
- Comparison against QSym, AFLFast, Angora, FairFuzz, AFL++
- 45% of fuzzing papers build on top of non-academic fuzzers
- 23% fail to compare against relevant state-of-the-art fuzzers or their won baseline

### Evaluation Setup

- 84/56% use a runtime of 24hrs, 40/27% less than 24hrs, 44/29% more than 24hrs, 8 did not specify runtime
- CPU cores: 38/25% did not specify the number of cores used, 40/27% used one core, 8% used two cores
- 111/74% allocated resources fairly (same number of cores, same runtime), from 23/15% of the papers the information could not be inferred, 8/5% did not evaluate other fuzzers. 5% unfairly allocated resources, mostly giving their fuzzers an advantage. Authors mostly did not give an explanation/motivation.
- Initial Seeds: 75/50% do not dsclose used seeds, 69/46% use same seeds for all fuzzers, 8/5% use diverging seeds, for the rest of the papers it is not clear.

### Evaluation Metrics
#### Coverage
- 115/77% use code coverage (29/19% use branch cov, 25/17% use edge coverage, 19/13% use block coverage, 8/5% use line coverage, 17/11% use unclear coverage measurements)
-- 67/45% lack clear definition of how they measured coverage
- 107/71% use number of (re-)discovered bugs
- 20/13% of the papers measured Time-To-Exposure
#### Known Bugs/New Bugs
- 59 papers claim one or more CVEs (found new bugs), 9.7 on average, 662 CVEs in total
- 339 CVS of claimned by 35 papers were investigated, only 145/43% were considered valid and have been fixed, 88/26% were marked as RESERVED, preventing analysis, 37/11% of the CVEs were invalid, 69/20% of the CVEs were ignored by the maintainers of the affected projects; many projects were abandoned, or have no widespread usage; maintainers ignore memory leaks in short running projects (e.g. assemblers)

#### Statistical Evaluation

- 63% of the works use no statistical test to assess their results,
- 15% use too few trials to achieve robust outcomes.
- 73% provide no measure of uncertainty.

#### Threat To Validity

Only 30 papers/20% of the papers contain a section on threats to the validity of the obtained results.

### Artifact Evaluation

#### MemLock / Artificial Runtime Environment and Unique Crashes

MemLock proposes to use memory usage as additional (to coverage) feedback to identify resource exhaustion bugs. Got "available" and "usable" badges

Observations:

- Stack Size was reduced to trigger stack overflow bugs
- MemLock depends on the number of unique crashes reported by AFL -> assume an inflated number of distinct bugs was reported.
- From reported 26 CVEs, up to five CVEs were requested for a single bug report
- For three targets AFL and Memlock were run, AFL found four bugs, MemLock three, although it repots a higher number of unique crashes
- On the reported CVEs refer to the same bug in mjs. Bug was never acknowledged by the mjs maintainers

#### SoFi / Exaggerated Vulnerabilities

SoFi aims to use a reflection-based analysis to create a syntactically and semantically valid, diverse set of seeds for fuzzing JavaScript engines

No artifacts were available. The source code of the paper authors was available, but incomplete. No reaction of the authors when asking for the missing parts.

Observation: All seven reported vulnerabilities (on SpiderMonkey and JavaScriptCore) are invalid and have been rejected by the developers of the respective projects.

#### DARWIN / Missing Baselines

Paper focuses on improved mutation scheduling. Authors propose to use an evolution strategy and dynamically adapt mutation selection on the target under test. Authors published artifacts on github.

Observations: 
- Coverage differences between DARWIN and baseline not statistically significant nor consistent with published results.
- Port implemented for MOpt may have erroneously restricted numberof usable mutations
- Incorrect version of AFL reported in the paper
- Artifact does not provide AFL 2.55b port for MOpt or their baseline AFL-S.
- Reported FuzzBench results in the paper could not be reproduced

#### FuzzJIT / Non-reproducible Measurements

FuzzJIT aims to detect bugs in JIT compilers, including those used in modern Browsers. The paper underwent artifact evaluation, got the "available" and "functional" badges.

Observations: 
- Reported coverage could not be reproduced
- Reported improvements of semantic correctness rate did not materialize
- Not possible to study the found bugs as important parameter were not provided

#### EcoFuzz / Uncommon Metrics

EcoFuzz proposes to replace AFL’s seed scheduling algorithm with a version 
relying on the adversarial multi-armed bandit model. This way, EcoFuzz finds more paths while generating less seeds. Goal: To find more paths with fewer seeds-

EcoFuzz has undergone artifact evaluation and was awarded the passed badge, indicating that the artifact is available and ready to be reproduced.

Observations:

- Paper does not report achived code coverage over time
- Reports the number of paths found
- Repetition of experiments using nm, libpng, objdump as targets
-- EcoFuzzer, achieves less code coverage in most cases

#### PolyFuzz / Unclear Documentation

Polyfuzz targets programs containing code in different languages, such as interpreter languages calling into native bindings.
PolyFuzz has been awarded the available badge.

Observations:

- Irregularities regarding the seed set used by PolyFuzz compared to other fuzzers

#### Firm-AFL / Incomplete Artifact

Firm-AFL, published at USENIX Security’19, aims to fuzz Linux-based IoT firmware via augmented process emulation. To do so, the core fuzzing loop targets a single binary under user-mode emulation, while selectively 
forwarding system calls to a full-system emulator.

Observations:

- Artifact was not available, but different versions of the source code were, in different repos.
- Due to lack of documentation, e.g. build instructions, re-using the artifact was difficult
- Baseline for coparison was AFL 2.06b, while the augmented code was using AFL 2.52b.

#### FishFuzz / Unfair Coverage Measurements

The paper proposes an input prioritization strategy based on a multi-distance metric that allows for optimizing the fuzzing efforts towards thousands of targets (e. g., sanitizer labels) in the sense of direct fuzzing.
FishFuzz has received the available and functional badges.

Observations: 
- FishFuzz was the only fuzzer to place coverage instrumentation not only within the actual target but also in the added ASAN instrumentation. Consequently, FishFuzz also stored inputs that exercised new coverage in the instrumentation;

### Revised Best Parctices for Evaluation

#### Reproducible Artifacts

- Make source code and documentation available as open source
- Recomment to participate in an artifact evaluation program
- Specify exact versions of targets and fuzzers used for comparisons
- Specify baseline on which the new implementation is based

#### Targets under Test

- Should be a representative set showing strengths of new approach which also allows comparisons with previous work
- If patches are applied to targets, this should be documented
- Restrictions on Fuzzers should be documented as well
- Usage of well-established benchmarks is recommended

#### Comparison To Other Fuzzers

- Compare against state of the art, and against baseline (if any)

#### Evaluation Setup

- Document used HW, experiment runtime, number of allocated cores, processes per fuzzer. Document how to reproduce the experiment
- Recommend to chose fuzzing for at least 24 hours
- Run experiments with an uninformed seed set or multiple seed sets
- Seeds must be available to support reproducibility, all fuzzers must have fair access to the seeds

#### Evaluation Metrics

- Use standardized, well-established metrics
- Use modern benchmarks
- Apply same coverage mechanism for all enmployed fuzzers
- For finding bugs in new targets, select reasonable ones (no inactive ones, ones that are insecure by design, ...)
- Use state-of-the-art fuzzers
- Crashes found by the fuzzers should be deduplicated manually before reporting them
- Do not request multiple CVEs for one and the same bug

#### Statistical Evaulation

- Run at least ten trials for statistical relevance
- Recommend to use alternatives to the Mann-Whitney-U test

--- 
## Slides Gliederung

### Slide 1: Paper Title, Date, Authors, Principal Content, presented when on which conference

### Slide 2: Recap on Fuzzing, Guidelines of Klees at al

### Slide 3: Methods
- 289 Papers, focus on fuzzing, in 2018-2023
- Randomly select 150 out of them
- Verify if recommendations were followed
- Out of 150 papers, 8 papers "that attracted intention" of the paper authors selected, examine artifacts, try to reproduce paper results
- Contacted authors of examined papers anonymously to ask for comments, ask for additional, missing, material

### Slide 4: Evaluation 1/2

- Reproducibility
- Targets under Test
- Evaluation against state-of-the-art
- Evaluation Test Setup

### Slide 5: Evaluation 2/2

- Metrics: Coverage, found known bugs, bound new bugs
- Statistical Evaluation
- "Threat to validity"

### Slide 6: Artifact Evaluation

- List the most significant findings, not mentioning the specific papers

### Slide 7: Revised Best Practices

- Enumerate new best practices
- Differences wrt Klees et al.

### Slide 8: "Our" Conclusion

- authors made their results publicly available on github
- see meta-review of conference 2024 IEEE Symposium on Security and Privacy (S&P)
- ... 