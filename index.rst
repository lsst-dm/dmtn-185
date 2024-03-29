..
  Technote content.

  See https://developer.lsst.io/restructuredtext/style.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. TODO: Delete the note below before merging new content to the master branch.

.. note::

   **This report is not yet published. Comments are welcome on this draft**

   Report of the Provenance Working Group (2020-2021)

.. Add content here.

Introduction
============

This document is the report of the Provenance Working Group charged through :cite:`LDM-722`.
The charge was to assess the provenance capabilities and uncaptured needs relating to the Gen3/Butler registry.

As the Working Group was constituted with representation from DM, EPO and SIT-COM, we quickly established that it was less than ideal to consider Gen3 provenance in isolation, and we widened the net to include related systems to ensure there were no gaps in the whole system's ability to record provenance of interest.
Ultimately we conducted a survey of provenance capabilities and needs across all systems that are involved in the kind of activities that will take place during commissioning and operations. As there is no equivalent document, we hope this report will be of use to the wider observatory and eliminate the need to have additional, system-specific Working Groups. The original scope of the charge (Gen3-related provenance) can be found in the section entitled `PipelineTask-level provenance <#_abyshwxrnm0j>`__.

Approach
========

We have divided provenance into a set of categories in order to facilitate the discussion. We define each one briefly below, and then organize our discussion of each category around four questions:

1. “What do we (want to) know?” (what information in this category can/should exist),
2. “What data paths do we have?” (how can we access this information from the system),
3. “What design do we have?” (what baselined or at least conceptual design for capturing and representing provenance information in this category exists), and
4. “What is the state of implementation?” (what work has already been done).

Following this, we provide a set of recommendations in each category, as well as a collation and overall recommendations in a concluding section.

We have endeavoured to be realistic about addressing this topic at a time where the project is already entering the Pre-Ops phase.
Instead of attempting to capture blank-slate requirements or exhaustive use-cases, we have taken the more pragmatic approach of auditing existing systems (or their design, where not complete), investigated what provenance information is captured (or capturable by them) and what gaps there are, either in provenance capture or in the ability of a system needing to
access provenance information to retrieve it.

Note that this document assumes that Rubin Observatory continues to have unbroken custody of the camera data, from the telescope, through its processing and to the eventual publication of it through the Science Platform to data-rights-holding astronomers. In the event that there is an interruption and/or processing of the data outside our software frameworks, the provenance chain could be severely compromised.


Working Group Committee
=======================

-  Amanda Bauer
-  Keith Bechtol
-  Gregory Dubois-Felsmann
-  Frossie Economou (chair)
-  Robert Gruendl
-  Simon Krughoff
-  Brian Stalder

Exposure-level provenance
=========================

By “exposure-level provenance”, we mean associating the raw image data with the global state of the observatory, as well as under what circumstances this observation was designed and executed.

What do we want to know?
------------------------

Given an exposure that was taken at the telescope, we want to trace: 

1. the state of the observatory during the observation,
2. the observing script (including version) that was executed that caused this observation,
3. the science program and field name (data taking campaign) that this observation was taken as part of (e.g., survey, commissioning run, etc.)
4. Any lower grouping that is meaningful but not derivable by the observing script - for example, a flat belonging to a group of flats taken with the purpose to co-add a master flat
5. Per-exposure ancillary data, eg. exposure quality flags/grades also should have its provenance recorded; for example, if an observer or processing system marks an exposure as "bad", who did it, why and when should be discoverable. 

What design do we have?
-----------------------

For (1) we have an extant design that allows that association of machine-readable data (specifically telemetry stored in the EFD) to be associated with an exposure using the start and end time of the exposure (see `DMTN-082 <http://dmtn-082.lsst.io>`__).

For (2) there is a discussion in \ `LTS-836 <http://ls.st/lts-836>`__ about how the observing scripts are processed by the observing queue.

For (3) the design is that the scheduler adds an observation to the queue specifying campaign parameters (eg, survey field name); these parameters are forwarded to the Camera Control System (CCS) which ensures that they are written to the file header. [see discussion at `dmtn-058 <https://dmtn-058.lsst.io>`__].

For (4), we understand the OCS queue implementation allocates a unique identifier for each OCS queue submission that can serve as such.

For (5) the logging working group produced a use-case for this [`LSE-490 <https://docushare.lsst.org/docushare/dsweb/Get/LSE-490/lse490_ElectronicLoggingSystemReport_rel1_20200925.pdf>`__], but it remains to be seen that this covers all situations where post-facto information is uncovered.


What data paths do we have?
---------------------------

For (1) the exposure table in the Consolidated Database (which contains the information that we conceptually think of as the ‘FITS header”) can be populated by using the stream-processed EFD tables in the Consolidated Database. The list of exposures used to construct this table can either be extracted from EFD events or it can be supplied during file ingest.

For (2) information about the script should be recordable in the EFD as it is available to DDS.

For (3) the information should be retrievable from the Butler registry, the EFD, and the exposure table.

For (4) the information is theoretically available from the Butler. 

For (5) exposure (human-entered) notes will be entered into a separate database, the access to this database and potential ways to status/flag exposures needs to be revisited.


What is the state of implementation?
------------------------------------

For (1) we are capturing the relevant telemetry in the EFD. We are not currently constructing the exposure table but this work is planned.

For (2) currently only the path to the observing script is being recorded and not generally retrievable.

For (3) this information is not currently in the header. This work is planned.

For (4) the Butler is planning on recording this information but this has not been tested yet.

For (5) this information is being designed, the write interface is currently being implemented on both the backend (OWL/OLE DB) and a front-end (LOVE).

Recommendations
---------------

The general approaches and notional designs seem reasonable, though there remain significant holes in the extant functionality. Following are the recommendations:

- [REC-EXP-1] As planned, program details known to the scheduler (such as science programme and campaign name) should be captured by the Butler.
- [REC-EXP-2] As planned, OCS queue submissions that result in meaningfully grouped observations should be identified as such in the Butler.
- [REC-EXP-3] Any system (eg. LOVE, OLE/OWL) allowing the entering or modification of exposure-level ancillary data should collect provenance information on that data (who, what, why).



Telemetry-level provenance
==========================

By “telemetry-level provenance” we mean associating observatory telemetry with properties of their originating systems (such as the name and version of a CSC) and allow their association with key observatory events (such as maintenance procedures).

What do we want?
----------------

We need to capture the instantaneous state of the system and what conditions it is operating in for situational awareness and to ensure appropriate and responsible scientific rigorousness in data recording.
This includes a complete picture of the states of all the subsystems, and the surrounding observatory environment (including the aspects of the visible sky, e.g. transmission, brightness).
For telemetry values, we would like to capture their origin, including properties (including software versions) of the CSCs (Commandable Software Components) that produced them. 

A separate record of maintenance and other changes in the hardware is made in a separate MMS (maintenance management system) database and should be retrievable by API to observatory reporting systems. 
Personnel notes and observations are recorded in the observatory-wide logging system.

What design do we have?
-----------------------

The EFD is designed to capture any time-series information accompanying telemetry values in a DDS topic (`SQR-29 <http://sqr-029.lsst.io>`__).
The Large File Annex (LFA) stores and archives larger (array) data files, such as all-sky camera images, webcam images (or movies), and input maps for the scheduler to be used in real-time or offline
analyses.

Both the Observatory Logging Ecosystem (OLE)  and MMS are still under design.

What data paths do we have?
---------------------------

Desired provenance data can be inserted and acquired via the SAL XML interface, eg. https://ts-xml.lsst.io/sal_interfaces/ATCamera.html#softwareversions
The LFA is implemented as a local S3 service on the summit and will be synced to the USDF at some cadence, and additional artifacts can be added to it.
The Camera Control System Database is a source of telemetry information, all of which is not published to the SAL and hence only a subset is captured in the EFD. 
Observatory human logging including operator comments (for both timely and offline annotation of images and miscellaneous temporal events), in a dedicated database, and shall be accessible via the Science Platform. 
Similarly, any hardware changes across the observatory are in principle recorded in a separate Maintenance Management System (MMS) database which is still under construction.


What is the state of implementation?
------------------------------------

The software architecture is mature and in production.
However, only a minority of CSCs publish all this information at this time.
More CSCs are being added all the time as we discover data gaps (e.g. seismic sensors, GIS, HVAC) and will likely continue into operations.
Any new CSCs should have provenance requirements explicitly stated (e.g. publishing their firmware version along with their telemetry) as makes sense for the CSCs in question. 

Data will be accessed by the users by multiple use-cases.

-  operators/engineers via LOVE (operator’s console), EUIs (engineer’s consoles), RSP notebook aspect, or Chronograf visualization interfaces.
-  scientists/external users via notebook aspect database access or butler if the associated telemetry is identified as critical information to an exposure
-  LFA data shall be accessible via RSP either through direct raw data access or via a specific butler or butler-like ingestion method if deemed necessary for the project and/or community.

Areas of concern focus on identifying all relevant aspects of the system and recording them in the EFD.
A standard way (salobj) of implementing CSCs has improved the process and templating and other ways of streamlining CSC implementation would help considerably in providing a robust provenance implementation.
Systems under evolving design (e.g. MMS, OLE/OWL) should explicitly address any provenance-related reporting requirements.

Recommendations
---------------

- [REQ-TEL-001] Investigate ways to expose all information in the Camera Control System Database to the EFD.
- [REQ-TEL-002] The MMSs should ideally have an API and at the very least a machine-readable export of data that would allow its data to be retrieved by other systems. 
- [REQ-TEL-003] Any new CSCs (and wherever possible any current CSCs that lack them) should have requirements on what provenance information they should make available to SAL so it can be associated with their telemetry. 


Software-level provenance
=========================


We define software-level provenance as the type of provenance information that:

1. Records the names and versions of the software that were participants in the system state of interest; “what were the camera readout parameters at the time this observation was taken” for example.
2. Could make these available in a way that would allow the system to be reconfigured back to that state.

Therefore within the scope of this section is data and metadata that would allow the reproduction of a previous state of the software systems of the observatory, including:

-  Software versions
-  Container versions
-  Software configuration
-  System configuration: e.g. voltages
-  Schema evolution management

   
What do we want?
----------------

In this section, we have drawn our examples from Data Management and the Telescope & Site groups as these are more familiar to the committee but our recommendations apply to all contributing software systems (including Camera, Facilities etc).

In these contexts, we want to know (and be able to reproduce) what telescope and instrument software versions were deployed when data taking occurred (such as wavefront sensing configurations, camera readout parameters, pointing models etc).

Similarly, we want to know the contributing code and dependencies that went into the production of a specific data product. 

What design do we have?
-----------------------

OSS-REQ-0122 specifies that the Data Management system will record the provenance of all its processing activities including software versions and hardware and operating system configurations used. 

LIT-151 requested that the above requirement not be limited to Data Management, but no action was taken. 

In some cases, we have developed software build/test/deploy chains that in practice guarantee a level of reproducibility (e.g. automated tagging of artifacts and a guarantee that the same tag cannot be applied to two different artifacts).

What data paths do we have?
---------------------------

Data paths to information that would lead to being able to recover a previous state of the system differs. Some examples are:


-  Software version

   -  DM -- All software is versioned via git and SHA1 hashes. There is also a release versioning system. The release versioning is not semantic.

   -  T&S -- All software is versioned via git and SHA1 hashes. Semantic versioning is applied.  With the person releasing the software determining whether to bump major, minor or patch release.  Follow git flow merge dev branch to default branch and tag.

-  Container versions

   -  DM -- Container images are produced and uploaded to a container repository like DockerHub. As with software, the containers have an associated unique hash so they can be identified. The Dockerfile used to produce the images is versioned via git, however, I’m unsure if there is a mechanism for matching up a given image with a git revision of a Dockerfile.

   -  T&S -- As in DM container images are uploaded to a container repository and images have a unique hash for identification. Docker files used in deployment are put in a single repository.  These are versioned using cycle versions rather than release versions.  The cycle is determined by SAL and salobj versions.

-  Software Configuration

   -  DM -- In DM, software configuration for the algorithms is handled by the configuration system of the pipeline tasks. This is discussed more in the PipelineTask provenance section. Configuration of many of the DM services is handled via a GitOps workflow mediated by the ArgoCD tool.

   -  T&S -- Configuration as code.  All configurations are git repos and versioned as code.  These are treated as code dependencies.

-  System Configuration

   -  DM -- For data processing, see PipelineTask-level Provenance Section.

   -  T&S -- The camera team takes care of the system configuration. We have not been able to determine what the extent of uncaptured configuration is for summit systems as a whole.

-  Schema evolution

   -  DM -- Schemas for the data products are stored in git and are versioned like other software. In some cases the build/test/deploy chains package the schema with software in containers, providing reproducibility through that route. In some cases schema for services are versioned by the avro/kafka schema migration machinery.

   -  T&S -- The message schemas are tightly controlled via XML documents that are versioned in git. They have a very strict release process that rolls out changes in the schema to running CSCs as a synchronized event. The Butler does not have a requirement to downgrade to previous schemas. 

Note that versioning in itself is not a sufficient guarantor of reproducibility.
For example, if some firmware does not have an embedded software version, or if that software version is manually updated, that can create situations where the same software version is assumed and/or reported, but in fact the code has changed.

What is the state of implementation?
------------------------------------

Some of these issues are being addressed by continuous improvements in build/test/deploy chains.

We are not aware of any tests that verify the ability to recover previous system states in most systems. 

Recommendations
---------------

- [REC-SW-1] There are a number of extant versioning mechanisms in DM and T&S software environments. Care should be to not proliferate those unreasonably but to share software versioning and packaging infrastructure where possible. As these systems are hard to get right, the more teams use them, the more robust they tend to be.

- [REC-SW-2] All systems should have individual explicit requirements addressing what, if any, demands there are to be able to recover a prior system state. When such requirements are needed, the systems should have to capture and publish in a machine-readable form, version information that is necessary to fulfil those requirements. Such requirements should cover the need for data model provenance, eg. whether it is necessary to know when a particular schema was applied to a running system. 

- [REC-SW-3] Software provenance support should include mechanisms for capturing the versions of underlying non-Rubin software, including the operating system, standard libraries, and other tools which are needed “below” the Rubin software configuration management system. The use of community-standard mechanisms for this is strongly encouraged.

- [REC-SW-4] Containerization offers significant and tangible advantages in software reproducibility for a modest investment in build/deploy infrastructure; it should be preferred wherever possible for new systems, and systems that predate the move to containerization should be audited to examine whether there is a reasonable path to integrate them to current deployment practices.



PipelineTask-level provenance
=============================

By PipelineTask provenance we mean information that is available in the Data Management middleware framework; PipelineTasks are the highest level building blocks from which data processing pipelines are constructed.


What do we want?
----------------

PipelineTask-level provenance is the finest grained provenance available through the LSST Science Pipelines without adding dedicated provenance-recording logic directly into the algorithmic code.
We believe this granularity is sufficient for reproducibility and traceability, and since the inputs and outputs are mediated by the Butler and all PipelineTasks are executed by core Gen3 code, robustness is high. 

This system will associate datasets, identified by DataId and type, and the collection in which they occur, with the PipelineTasks that produced them, identified by name and class, and the as-executed values of their configuration objects.

The system records that a given input was presented to a PipelineTask, not that the data in that input was actually used in the generation of the final result (e.g., it might fail a quality cut and not in fact be included in a coadd). This is the correct approach in order to achieve reproducibility of previously executed pipeline steps. 

Additionally, it appears *(needs confirmation)*\ that as-executed lists of package versions, and physical dataset locators *(URIs?)* are recorded by the command-line activator (pipetask in ctrl_mpexec).

Provenance capture
^^^^^^^^^^^^^^^^^^

For a given output dataset of a PipelineTask we want to capture:

1. The specific versions of the PipelineTask stack that were run to create it;
2. The computing environment within which it was run;
3. The specific configuration (pex_config) that was applied, after the “stacking up” of all defaults and overrides;
4. The input datasets presented to the PipelineTask that generated the output, ideally named in both site-independent (DataID) and physical forms (URIs);
5. Any QA metrics that were generated “in situ” as part of the calculational work of the PipelineTask (see Metrics-Level Provenance)
6. Logs and/or other outputs to indicate success/failure performance, etc. (see Log-Level Provenance)

For (4), we want the URIs in order to be able to disambiguate between eg. data products that have been produced at different Data Facilities with the same computed DataIDs. 
   
Provenance utilization
^^^^^^^^^^^^^^^^^^^^^^

We want to be able to perform queries against the recorded provenance, such as “tell me which raws or which calexps contributed to this coadd” from the Butler (see figure for a visual aid).

The above capture and query capability is reflected in DMS-MWBT-REQ-0094 & DMS-MWBT-REQ-0095 (`LDM-556 <http://ldm-556.lsst.io>`__) and ultimately flows down via LSE-61 :cite:`LSE-61` from LSE-30 (OSS-REQ-0122) which requires that sufficient provenance is recorded that data products can be reproduced.

We would like to have both code and command-line support for the operation “re-run, as exactly as possible, the processing that was used to generate dataset X”, based on stored provenance.
This would, for instance, use the frozen “as-executed” configuration values as a 100% override to any default configuration values in the code used for the re-run.
This re-run capability is needed for validation as well as for use in “virtual data product re-creation” services.
It will also be needed by Notebook Aspect users.

Additionally, we would like a provenance web service to allow Science Platform users to perform these queries, such as the IVOA provenance ProvDAL service.

We are not aware of any work that has been done to date on mapping the PipelineTask provenance to common community three-term ontologies for provenance such as the W3C or IVOA provenance models. However, the information content seems likely to have a fairly natural mapping.

What design do we have?
-----------------------

`LDM-152 <http://ldm-152.lsst.io>`__ specifies that the configuration and inputs to PipelineTasks are preserved.


Task-level provenance has previously been discussed in `DMTN-083 <http://dmtn-083.lsst.io>`__ but it predates the PipelineTask design and some sections have been obsolesced by the current baseline.


What data paths do we have?
---------------------------

The Science Pipelines executor currently records software versions and configuration in the Butler.
In the design, the executor stores the quantum graph in the Butler in a form that would allow an API to service the example queries above.

What is the state of implementation?
------------------------------------

From the list above, (1) and (2) are stored and queryable by the Butler API while (3) is not yet implemented but is planned.

VO access to this information via ProvDAL is not planned in construction.

Recommendations
---------------

- [REQ-PTK-001] As planned, complete the recording of as-executed configuration for provenance.

- [REQ-PTK-002] As planned, complete the storage of the quantum graph for each executed Pipeline in the Butler repository.
  
- [REQ-PTK-003] Code and command-line support for recomputing a specified previous data product based on stored provenance information should be provided.

- [REQ-PTK-004] A study should be made on whether W3/VO provenance ontologies are a suitable data model either for persistence or service of provenance to users. 

- [REQ-PTK-005] URIs (as well as DataIDs) should be recorded in Butler data collections.



Workflow-level provenance
=========================

Note that in our architecture, some of the provenance use cases that are typically the domain of the workflow system, specifically software version provenance, are handled by PipelineTask-Level provenance.
This includes both pipeline software versions and third-party package versions and is an effect of the design where there are elements of the Science Pipelines (specifically  pipe_base) that is “upstream” of the workflow system, as it generates the quantum graph submitted to the workflow.

Similarly, as opposed to some systems where a directed acyclic graph is described in some workflow specific language (or translated from the common workflow language), the source of primacy is the quantum graph computed by the pipeline task framework itself.

The low-level workflow system must be able to report details about how the quantum graph was executed.
Specifics are enumerated in the recommendations.

`LSE-30 <http://ls.st/lse-30>`__ does require operating system and
hardware provenance to be recorded. This could be done at workflow-level provenance, but given the lack of requirement at this level it might be simpler to just add this information to PipelineTask-level provenance (where the OS is already recorded but not the version).

Recommendations
---------------

- [REQ-WFL-001] Logs from running each quantum must be captured and made available from systems outside the batch processing system.

- [REQ-WFL-002] Any workflow level configuration and logs must be persisted and made available from systems outside the batch processing system.
  This information should be associatable with specific processing runs.

- [REQ-WFL-003] Failed quanta must be reported including where in the batch processing system the quantum was running at the time of failure.

- [REQ-WFL-004] Though no requirement exists, it should be possible to inspect, post-facto, the resource usage (CPU, memory, I/O etc.) for individual workers.

- [REQ-WFL-005] Both the OS and the OS version must be recorded.
  This requirement may be met within the pipeline task provenance, but it is an upscope since currently, only the OS type is recorded.

File-level provenance
=====================

We define file-level provenance as the inputs that contributed to the production of that data, including other files and software.
There are various ways of representing these, e.g. a graph of predecessor data.
By tracing a provenance chain one can then reconstruct the relationship of products to upstream or downstream products and processes.

An alternative means to express provenance would take the form that associates a collection of inputs and outputs, along with a record of a broader pipeline task and configuration.
The granularity of such provenance is not amenable to answering questions about how a product
was used without *a priori* knowledge of the pipeline processing, but can be much faster for certain search operations. 

Both the above cases can be thought of as an extrapolation of PipelineTask- and Workflow-level provenance to the file level.
The two cases are not mutually exclusive (ie. they could both be persisted).
In fact, the methods for exploiting the information can be left to the users, so long as the relational information is systematically stored.

What do we want?
----------------

There are two relevant requirements in `LDM-556 <http://ldm-556.lsst.io>`__:

1. Persisting provenance information with the raw data IDs that contributed to a dataset into the final export data format (be it FITS or alternative) (DMS-MWBT-REQ-0093)
2. Same but with the immediate parents (DMS-MWBT-REQ-0093)

What design do we have?
-----------------------

There is no current design for implementing this. Three options would be:

- “Burning it” into the file on write (on Butler Put)
- Packaging it with the file on read/export (by the service publishing the file)
- Saving relational information in the Butler registry and leaving the methodology for its retrieval/use/exploitation to the user.

An alternative to this approach would be to fulfil the spirit of the requirement by burning into the file a service call (eg. DataLink) that supplies the required provenance information.
Metadata such as the run collection, dataId, and dataset type are not (currently) stored in persisted formats.

The filename should not be relied on for provenance lookup since it may be changed by the user and furthermore, the filenames alone cannot be relied on because they are not unique to a specific processing attempt of a given product.

Finally, it is often NOT desirable to express all parent files that ever led to the creation of a data product as part of that product.
For example, recording every flat field that was used in the generation of a CalExp that in turn was used as part of a COADD image would be wasteful.
The record of such relations is better stored in a database (eg. Butler registry) where it can be queried than accumulated/persisted in the header of each output image.
The unanswered question is whether there are cases where such file level provenance information should be saved in an image header.

What data paths do we have?
---------------------------

The information is known as part of the PipelineTask-Level provenance above.

What is the state of implementation?
------------------------------------

Not currently implemented.

We are concerned that data processing and imminently data-taking is underway prior to a system to record this provenance information is in existence. 

Recommendations
---------------

-  [REC-FIL-1] Serialised exported data products (FITS files in the requirements) should include file metadata (e.g. FITS header) that allows someone in possession of the file to come to our services and query for additional provenance information for that artifact (e.g. pipeline-task level provenance).

- [REC-FIL-2] A study should be made of the possibility of embedding a DataLink or other service pointer in the FITS header in lieu of representing the provenance graph in the file.

- [REC-FIL-3] Irrespective of ongoing design discussions, every attempt should be made to capture information that could later be used to populate a provenance service. 


Source-level provenance
=======================

By source-level provenance we mean astronomical sources in catalogs (sources, objects, etc). For simplicity we use "Source ID" in this section to mean the appropriate identifier of any source-like product (DIAsource, DIAObject, Object, etc)

What do we want?
----------------

We agree with `DMTN-085 <http://dmtn-085.lsst.io>`__ (report of the QA working group) that there is no strong requirement for pixel-level per-source/object provenance beyond an association with the dataset from which the source measurement was derived since we are no longer using the multifit approach (and its simultaneous source model-fitting approach).

However, there are per-source metadata that need to be propagated to the final data release product.
The two that we have identified are flags and footprints


Flags include boolean information about the source detection quality, e.g., were there saturated pixels in the detection.
Flags can also be used to capture processing information such as which objects were used for astrometric calibration, photometric calibration, PSF modelling, and whether a source is an injected fake. 

A footprint identifies which pixels were used to compute measurements on the source/object.
Because current deblending algorithms may distribute flux from a single pixel among multiple footprints, there are actually two types of footprint:

- Per source/object heavy footprints (pixel indices as well as flux values)
- Per source/object (lightweight) footprints (pixel indices only).

Pixel-level mask flags can be retrieved using an individual footprint.


What design do we have?
-----------------------

Source-level provenance has previously been discussed in `DMTN-083 <http://dmtn-083.lsst.io>`__ but it predates the Gen3 Butler design and some sections have been obsolesced by the current baseline.

The DPDD explicitly allows up to 64 bits for source flags and 128 bits for object flags.
Footprints are not enumerated by the DPDD, although it is assumed that they will be provided in some form with our catalogs. 

What data paths do we have?
---------------------------

The Source ID encodes certain provenance information, including having 4 bits available to associate a source with a specific Data Release.
This means that only 16 Data Releases can be recorded.
The Source ID by itself does not encode any provenance information relating to a specific (re-)run; this information is available in the collection created by that (re-)run. 
Similarly for the ObjectID. 

Provenance for flags and footprints is accessible via the Source ID associated with that footprint or flag.

Our source fitting algorithm (Scarlet) is deterministic; in any situation where an algorithm with a (for example random) seed is used, the seed should be preserved in the provenance metadata.

We also have some data that is smaller than a CCD but bigger than a source, such as healpix-mapped seeing data.
We have not considered here the provenance needs of such aggregated synthetic data. 

What is the state of implementation?
------------------------------------

Source/Object IDs are being generated, although it is not clear to us whether:

1. They are compliant with what the DPDD describes
2. Whether the 64-bit sourceIDs specified in DPDD are sufficient 

Measurement algorithms produce flags and footprints already.

The DPDD specifies 64 bits for source flags and 128 bits for object flags.
We are not aware of an analysis that confirms that these are sufficient.

Though the footprints are computed as part of processing and are persisted as intermediate products, there is no implementation for providing them to end-users (they are available directly through the butler in gen 3).

Though heavy footprints are included in the sizing model, there is only passing mention of them in the DPDD.

Recommendations
---------------

- [REC-SRC-001] Perform a census of produced and planned flags to ensure that 64 bits for sources and 128 bits for objects are sufficient within a generous margin of error. This activity should also be carried out for DIASources and DIAObjects source IDs.

- [REC-SRC-002] We are concerned that merely encoding a 4-bit data release provenance in a source does not scale to commissioning needs and the project should decide whether it is acceptable for additional information beyond the source ID to be required to fully associate a source with a specific image.

- [REC-SRC-003] More generally, a study should be conducted on whether 64 bit source IDs are sufficient.

- [REC-SRC-004] Although not provenance-related, we recommend that the DPDD be updated to clearly state whether footprints and heavy footprints are to be provided.


Metrics-level provenance
========================

In this document, “metrics” refers to persisted performance indicators quantifying the technical and/or scientific evaluation of a unit of scalar data or computational process related to the Science Pipelines and/or derived data products.

What do we want?
----------------

The metrics framework (lsst.verify) specifies a need for provenance information for two purposes:

1. Identify uniquely a production run (job ID) that resulted in a metric measurement having been produced
2. Associate metric measurements with provenance information that allows for meaningful comparisons (e.g., that they derive from data processing runs taken with the same instrument, same filter; that they from a particular visit, etc.)

See `SQR-019 <http://sqr-019.lsst.io>`__ for more discussion. 

What design do we have?
-----------------------

The original baseline assumed that there would be a workflow-level provenance system to provide (1) and (2).
With the advent of the Gen3 Butler and the task-level provenance model, the needed information can largely be derived.

The QA Strategy Working Group (`DMTN-085 <https://dmtn-085.lsst.io/>`__) makes several specific recommendations related to the calculation, persistence, and dissemination of metrics.

-  The computation, selection, and aggregation steps that define a metric should be cleanly encapsulated
-  Metric values should be stored with complete provenance granularity (source, CCD, patch, dataset)
-  Metric values should have Butler dataIds and the Data Butler should be usable to persist and retrieve metric values
-  Formalise the lsst.verify.metrics system as the source of truth for metric definitions

The association of metrics with Butler dataIds and storage of metrics using the Data Butler are significant steps towards the two goals above.

We anticipate that metrics (in the more general sense of derived scalars) will also be generated from other types of data besides the Science Pipelines and derived data products, for example, metrics derived from telemetry and the state of the system, as well as measures
of survey progress and other compound metrics.
SQuaSH is the de facto system for curating such metrics. 

What data paths do we have?
---------------------------

Butler has a concept of a “run” as in a “run collection” - a group of datasets that hold the outputs of an execution run (job).
The identifier of this run collection is passed in as an argument to the workflow system.
This can serve as a job ID for the metrics system; however note that it is up to the submitter to ask for a unique job ID (as opposed to, for example, a workflow system like Jenkins where a job is submitted and the system assigns the job ID).
For a further discussion of policies for collection names, see `DMTN-167 <http://dmtn-167.lsst.io>`__ .

The Butler team is planning for the low-level executor for pipeline tasks to generate a unique identifier for a pipeline execution run, which effectively can be used as the "job ID" initially envisaged.

Given a run identifier, the Butler will be able to be queried for other information pertinent to the run, such as the instrument the processed data originated from.

What is the state of implementation?
------------------------------------

Previously, the metrics framework used a basic shim for provenance information.
Leveraging the emerging capabilities of the Gen3 Butler addressed the need for that shim. 
Storing metrics as Butler ad-hoc dataset types allows metrics to be directly persisted in the run collection with the associated data they were derived from.
Specifically, a Butler repo can hold lsst.verify.Measurement objects in collections.
When tasks that compute metrics put the lsst.verify.Measurement back into butler, we fulfil most of the provenance goals in this area.
(This approach is used, for example, by the faro metrics calculation software.)
An advantage of this approach is that the configuration information used for the execution is also stored in the Butler repo.

Storing metrics in the Butler as ad-hoc datasets significantly limits the usability and utility of these metrics. If the metrics were supported  as a native structured Butler dataset, then we would be able to

1. Query the Butler for what metrics are available (metrics discovery)
2. Have the ability to filter other Butler queries on the basis of metric measurements
3. Significantly increase the robustness of metric transport to Squash by associating the lsst.verify metrics specification with the Butler 

We understand such development is not planned in construction. 
   
Recommendations
---------------

- [REC-MET-001] For metrics that can be associated with a Butler dataId, the metrics should be persisted using the Data Butler as the source of truth. The dataId associated with the metric should use the full granularity.
- [REC-MET-002] Any system that uses Butler data to derive metrics should persist them in the Butler provided that the metrics are associable with a Data ID.
- [REC-MET-003] When lsst.verify.Job objects are exported, the exported object should include the needed information (run collection and dataId) to associate with the source of truth metric persisted with the Data Butler.
- [REC-MET-004] A plan should be developed for persisting metrics that are not directly associated with Butler-persisted data.
- [REC-MET-005] Even if effort for implementation is not available in construction, we should develop a conceptual design for structured, semantically rich storage of metrics in the Butler.


Log Provenance
==============

What do we want?
----------------

Logs, i.e., machine-generated output from software and systems involved in data-taking are sometimes necessary in order to understand unexpected behaviour.
Log provenance shares most provenance requirement with metrics data, except for being a blob rather than a scalar. 

What data paths do we have?
---------------------------

As far as it is known, services and software systems log to STDOUT.
Following RFC-767 we expect these to be timestamped in UTC where they are not already. 

What is the state of implementation?
------------------------------------

There is a patchwork of implementation in terms of curating and making searchable logs.
Some systems do not dispatch to a central service, some do so to the summit's greylog system, some send to an ELK cluster at the IDF.
There does not seem to be currently a centralised system for dispatching all eg. LDF logs to a single LDF log management cluster. 

Recommendations
---------------

- [REC-LOG-1] Since time is the primary provenance element for a log entry, systems are to produce (or make searchable) in UTC.

- [REC-LOG-2] Each site (summit, IDF, USDF, UKDF, FRDF) should provide a log management solution or dispatch to another site's log management service to aid log discoverability.

- [REC-LOG-3] Individual systems should make clear log retention requirements. 



.. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
    :style: lsst_aa
