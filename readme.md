# Hypercerts on BOS

Built using Hyperfiles: https://github.com/flowscience/hyperfiles

### **About**

An implementation of Hypercerts using Near Protocol's Blockchain Operating System (BOS). Hypercerts on BOS (HoB) is a decentralized front-end stored entirely on-chain that interacts with both the Near socialDB contract as well as the Hypercert contract on Optimism to enable minting on either network. The entire application can be integrated into any BOS gateway deployment such as near.org, near.social, and opencann.net with a single line of code. BOS native Hypercerts leveraging a standardized type schema. Standardized fields can make Hypercerts composable with other implementations of impact certificates (e.g. to aggregate & analyze metadata like scope, dates, contributors, etc). 

Docs
[https://www.notion.so/opencann/Hypercerts-on-BOS-a779c2abf7e54e4bb5782e569dd14419](https://www.notion.so/Hypercerts-on-BOS-a779c2abf7e54e4bb5782e569dd14419?pvs=21)

Demo
https://near.org/flowscience.near/widget/create.hypercert

Built during Funding The Commons, Public Goods Hackathon (Aug 2023)
https://app.buidlbox.io/projects/hypercerts-on-bos

---

https://www.youtube.com/watch?v=uu6ABkJSqsI

## Core Types

### [metadata](https://everything.dev/every.near/widget/every.type.create?typeSrc=every.near/type/metadata): redundant fields applicable to any type or thing

- description (s)
- image (s)
- backgroundImage (s)

### [file](https://everything.dev/every.near/widget/every.type.create?typeSrc=flowscience.near/type/file): a collection of records

- fileName (str, s)
- fileFormat (thing, s)
- records (thing, m)
- owner (str, s)
- permissions (thing, s)

### [fileformat](https://everything.dev/every.near/widget/every.type.create?typeSrc=flowscience.near/type/fileformat): standard types for files

- formatName (str, s)
- fields (thing, m)
- ~~fileNames (thing, m)~~

### [field](https://everything.dev/every.near/widget/every.type.create?typeSrc=flowscience.near/type/field): primitives that compose into fileFormats

- fieldName (str, s)
- primitive (str, s)
- ~~fileFormats (thing, m)~~

### [record](https://everything.dev/every.near/widget/every.type.create?typeSrc=flowscience.near/type/record): individual pieces of data (a datum)

- recordName (str, s)
- fields (str, m)
- recordValues (str, m)
- recordOwner (str, s)
- isEncrypted (bool, s)
- permissions (thing, s)

### permissions:

- TBD
- sking.near/widget/DAO.Permissions

## Core fileformat Types

These enable files to be used for decentralized compute over Bacalhau via PLEX.

[See example Colab notebook for off-chain demo.](https://colab.research.google.com/drive/1wHWzHR6zHYUm3RsPJ2pJh2t2WgLti34E?usp=sharing)

### job: an “.io file” to specify job type and input/output dir

- jobName (str, s)
- jobType (thing, s)
- inputfiles (thing, m)
- CID (str, s)
- owner (str, s)

```json
[
  {
    "outputs": {
      "outdata": {
        "class": "File",
        "filepath": "testdata_fastqc.zip",
        "ipfs": "QmXXKpKz2w1b992MXpSo3Dmohu46d4V4Y2WXLQZtWa9AKR"
      },
      "report": {
        "class": "File",
        "filepath": "testdata_fastqc.html",
        "ipfs": "QmWw6oevnbPqxJGVBMmAVPsRCfHEHcf8D6TFxThXENdx1b"
      }
    },
    "tool": {
      "name": "fastqc",
      "ipfs": "QmRsqab1yXAc8GPipkXkBjTjJq7VpXSRv5qdthLymm4Mzm"
    },
    "inputs": {
      "reads": {
        "class": "File",
        "filepath": "testdata.fastq",
        "ipfs": "QmUvALPCVL5ZgnRWyvfqjrTSK9UgLW1JJTLsisFooRpcGD"
      }
    },
    "state": "completed",
    "errMsg": "",
    "userId": "",
    "bacalhauJobId": "0581f011-8070-4ddb-959b-f6bc2c43e2ed"
  }
]
```

### jobtype: standard types for jobs

Is this redundant with toolconfig???

- jobType (str, s)
- toolConfigFile (thing, s)
- jobFiles (thing, m)

### toolconfig: specify container path and jobtype parameters

Enforce input file formats are of the appropriate type (so the job will work).

Designate output file formats to enable composability in pipeline builder.

- configName (str, s)
- inputfileFormats (thing, m)
- outputfileFormats (thing, m)
- class
- name
- description
- baseCommand:

[Tested working toolConfigFiles can be found on Github (see example below).](https://github.com/Open-Cann/plex/tree/main/tools)

https://github.com/Open-Cann/plex/blob/main/tools/fastqc/fastqc.json

```json
{
  "class": "Tool",
  "name": "fastqc",
  "description": "Comprehensive quality control tool for high-throughput sequence data",
  "doi": "https://doi.org/10.48550/arXiv.2202.05146",
  "baseCommand": ["/bin/bash", "-c"],
  "arguments": [
    "fastqc $(inputs.reads.filepath) --outdir=/outputs/"
  ],
  
  "dockerPull": "staphb/fastqc:0.12.1@sha256:f5d8f72753269e0cee071fe198c89a59a1f8071445739b3398f7818f7cb039ae",
  "gpuBool": false,
  "networkBool": false,

  "inputs": {
    "reads": {
      "type": "File",
      "glob": ["*.bam", "*.sam", "*.fastq"]
    }
  },

  "outputs": {
    "report": {
      "type": "File",
      "glob": ["*.html"]
    },      
    "outdata": {
      "type": "File",
      "glob": ["*.zip"]
    }
  }
}
```

```json
{
  "class": "CommandLineTool",
  "name": "equibind",
  "description": "Docking of small molecules to a protein",
  "baseCommand": ["/bin/bash", "-c"],
  "arguments": [
    "mkdir -p /tmp-inputs/tmp;",
    "mkdir -p /tmp-outputs/tmp;",
    "cp /inputs/* /tmp-inputs/tmp/;",
    "ls /tmp-inputs/tmp;",
    "cd /src && python /src/inference.py --config=/src/configs_clean/bacalhau.yml;",
    "mv /tmp-outputs/tmp/* /outputs/;",
    "mv /outputs/lig_equibind_corrected.sdf /outputs/$(inputs.protein.basename)_$(inputs.small_molecule.basename)_docked.$(inputs.small_molecule.ext);",
    "mv /tmp-inputs/tmp/*.pdb /outputs/;"],
  "dockerPull": "ghcr.io/labdao/equibind:main@sha256:21a381d9ab1ff047565685044569c8536a55e489c9531326498b28d6b3cc244f",
  "gpuBool": false,
  "networkBool": false,
  "inputs": {
    "protein": {
      "type": "File",
      "item": "",
      "glob": ["*.pdb"]
    },
    "small_molecule": {
      "type": "File",
      "item": "",
      "glob": ["*.sdf", "*.mol2"]
    }
  },
  "outputs": {
    "best_docked_small_molecule": {
      "type": "File",
      "item": "",
      "glob": ["*_docked.sdf", "*_docked.mol2"]
    },
    "protein": {
      "type": "File", 
      "item": "",
      "glob": ["*.pdb"]
    }
  }
}
```

## Lists of Things

### metadata

- every thing else has a metadata thing associated with it already, right???
    - or is metadata stored in the socialDB contract?
    - https://near.org/efiz.near/widget/every.type.metadata

---

### files

Core

- every.file
- every.fileformat
- every.record
- every.field
- every.job
- every.jobtype
- every.toolconfig
- {accountId}/thing/impact.file

Featured

- every.hypercert

---

### fileformats

- [hypercert](https://everything.dev/mob.near/widget/WidgetSource?src=flowscience.near/thing/fileformat.hypercert)

[fileformat.hypercert](https://www.notion.so/fileformat-hypercert-c2c11531d8ed448798c1e2c6dc17a650?pvs=21)

- impact

[fileformat.impact](https://www.notion.so/fileformat-impact-dc0014edfb2c474e84bcf7230a640d54?pvs=21)

- fileformat.job
- fileformat.jobType
- fileformat.toolConfig

types = fileformats = types???

[type:artist](https://www.notion.so/type-artist-8f301fbec3894f1c96f7439676c0dc68?pvs=21)

[type:DAO](https://www.notion.so/type-DAO-b614e064922d4bd48d6dd329d42bfa5a?pvs=21)

---

### fields

- [hypercertName](https://everything.dev/mob.near/widget/WidgetSource?src=flowscience.near/thing/field.hypercertName) (string, s)
- [hypercertProject](https://everything.dev/mob.near/widget/WidgetSource?src=flowscience.near/thing/field.hypercertProject) (url, s)
- [hypercertScopeTags](https://everything.dev/mob.near/widget/WidgetSource?src=flowscience.near/thing/field.hypercertScopeTags) (list, s)
- [hypercertWorkStart](https://everything.dev/mob.near/widget/WidgetSource?src=flowscience.near/thing/field.hypercertWorkStart) (date, s)
- [hypercertWorkEnd](https://everything.dev/mob.near/widget/WidgetSource?src=flowscience.near/thing/field.hypercertWorkEnd) (date, s) - how to make indefinite? (leave empty)
- [hypercertContributors](https://everything.dev/mob.near/widget/WidgetSource?src=flowscience.near/thing/field.hypercertContributors) (list, s)
- [hypercertUsageRights](https://everything.dev/mob.near/widget/WidgetSource?src=flowscience.near/thing/field.hypercertUsageRights) (str, s)
- [hypercertAllowlist](https://everything.dev/mob.near/widget/WidgetSource?src=flowscience.near/thing/field.hypercertAllowlist) (list, s)
- [hypercertDistribution](https://everything.dev/mob.near/widget/WidgetSource?src=flowscience.near/thing/field.hypercertDistribution) (int, s) - how to set deduplicate = TRUE???
- description (string, s)
- image (img, s)
- backgroundImage (img, s)

---

### records

- hypercert.1
- file.1
- job.1

---

### jobs

- TBD

---

### jobTypes

- TBD

---
