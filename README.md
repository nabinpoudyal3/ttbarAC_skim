# ttbarAC_skim

Skim code setup for producing Ntuples for ttbar charge asymmetry analysis.  
The framework is setup for running at the LPC, and saving ntuples to the LPC Tier3.


## Getting Started

Check out CMSSW release:
```
cmsrel CMSSW_9_0_1
cd CMSSW_9_0_1/src
cmsenv
git cms-init
```

Check out [BESTProducer](https://github.com/justinrpilot/BESTAnalysis) and other packages:
```
# VID
git cms-addpkg RecoEgamma/ElectronIdentification
git cms-addpkg PhysicsTools/SelectorUtils
# BEST 
git clone git@github.com:justinrpilot/BESTAnalysis -b 90x_prod
# LWTNN
mkdir lwtnn && cd lwtnn/
git clone https://github.com/demarley/lwtnn.git
cd lwtnn/
git checkout CMSSW_8_0_X-compatible
cd ../../
# Analysis code -- now linked with this package
mkdir Analysis
git clone https://github.com/cms-ttbarAC/CyMiniAna.git Analysis/CyMiniAna

git clone git@github.com:cms-ttbarAC/ttbarAC_skim

scram b -j8
```

## Ntuple Production

Instructions for producing flat Ntuples with BEST tagger inputs.

```
cd ttbarAC_skim/ttbarAC_skim/test
voms-proxy-init -voms cms
source /cvmfs/cms.cern.ch/crab3/crab.csh
```

To submit crab jobs, the script `runSkim.py` is processed for MC using the `cmsRun` executable.

First, edit `crab_*.py` with a new dataset name to reflect the sample you are processing, e.g., `crab_SMttbar.py`.  
Make sure to the output directory is pointed to `/store/group/lpctop/edmNtuples/`.  
If you are processing data, modify the argument `config.JobType.pyCfgParams = ['isMC=True']` to `config.JobType.pyCfgParams = ['isMC=False']`.  
To submit the CRAB jobs, enter the command:
```
crab submit -c crab_*.py --dryrun
crab proceed
```
You can also submit the crab jobs with only `crab submit -c crab_*.py`, where `crab_*.py` is the crab configuration file.  
Monitor the CRAB jobs and resubmit any failed jobs.


## Previous setup (still possible but not recommended)

### EDM Production:

Instructions for producing EDM Ntuples with BEST tagger inputs and then flat trees for analysis.

```
cd ttbarAC_skim/ttbarAC_skim/test
voms-proxy-init -voms cms
source /cvmfs/cms.cern.ch/crab3/crab.csh
```

To submit crab jobs, the script `runSkim.py` (`runSkim_data.py`) is processed for MC (data) using the `cmsRun` executable.

First, edit `crab_*.py` with a new dataset name to reflect the sample you are processing, e.g., `crab_SMttbar.py`.  
Make sure to the output directory is pointed to `/store/group/lpctop/edmNtuples/`.  
To submit the CRAB jobs, enter the command:
```
crab submit -c crab.py --dryrun
```
either remove `--dryrun` to submit the jobs, or enter `crab proceed` after running the above command.  
Monitor the CRAB jobs and resubmit any failed jobs.


### Flat Ntuple Production

After crab jobs are finished, make a text file with the filenames (starting with `/store/group/...`):

```
xrdfsls -u /store/group/lpctop/ttbarAC/edmNtuples/.../ | grep "ana" > textfile.txt
```
If there are multiple EDM files in this directory, change `ana` to a specific keyword in the desired files.  
Then, setup the condor scripts to submit jobs to the LPC condor system that process the EDM ntuples into flat ntuples.  
The condor scripts to check are `submit.sh`, `analyze.sh`, and `treeMaker_fwlite.py`.  
There shouldn't be any changes necessary in `treeMaker_fwlite.py` or `submit.sh`, but `analyze.sh` may need changes for the following two lines:
```
output_path="/store/group/lpctop/ttbarAC/flatNtuples/"
...
python treeMaker_fwlite.py --files ${file} --isMC 1
```
where `output_path` may need to be changed, and `isMC` needs to be 0 for data samples.  

To submit the jobs, enter:

```
./submit.sh files.txt <process>
```

Where `<process>` is the name of the process, e.g., `SMttbar`.  
Note: 
> this string gets added to the ROOT output files, 
> so you use that same string later with `do_hadd.sh` 
> to grab only the files from a certain process and `hadd` them

Monitor these jobs using `condor_q ${USER}` or using [landscape](https://landscape.fnal.gov/lpc/dashboard/db/lpc-summary?orgId=1).  
Once the jobs are finished, flat ntuples are available for analysis using [CyMiniAna](https://github.com/cms-ttbarAC/CyMiniAna).

## Questions
Contact the authors.
