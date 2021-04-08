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
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. TODO: Delete the note below before merging new content to the master branch.

.. note::

   **This technote is not yet published.**

   Report of the Provenance Working Group (2020-2021)

.. Add content here.

Introduction
============

This document is the report of the Provenance Working Group charged through `LDM-722 <https://ldm-722.lsst.io>`__.
The charge was to assess the provenance capabilities and uncaptured needs relating to the Gen3/Butler registry.

As the Working Group was constituted with representation from DM, EPO and SIT-COM, we quickly established that it was less than ideal to consider Gen3 provenance in isolation, and we widened the net to include related systems to ensure there were no gaps in the whole-system’s ability to record provenance of interest.
Ultimately we conducted a survey of provenance capabilities and needs across all systems that are involved in the kind of activities that will take place during commissioning and operations. As there is no equivalent document, we hope this report will be of use to the wider observatory and eliminate the need to have additional, system-specific Working Groups. The original scope of the charge (Gen3-related provenance) can be found in the section entitled `PipelineTask-level provenance <#_abyshwxrnm0j>`__.

Approach
========

We have divided provenance into a set of categories in order to facilitate the discussion. We define each one briefly below, and then organize our discussion of each category around four questions:

1. “What do we (want to) know?” (what information in this category can/should exist),
2. “What data paths do we have?” (how can we access this information from the system),
3. “What design do we have?” (what baselined or at least conceptual design for capturing and representing provenance information in this category exists), and
4. “What is the state of implementation?” (what work has already been done).

Following this we provide a set of recommendations in each category, as well a collation and overall recommendations in a concluding section.

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
2. the observing script (inc version) that was executed that caused this observation,
3. the science program and field name (data taking campaign) that this observation was taken as part of (e.g., survey, commissioning run, etc.)

Then the next obvious state we need to trace at the exposure level is:

1. any subsequent discoveries about the data quality that are deemed relevant (e.g., a previously undocumented event/condition, a new QA metric calculation, etc.)
2. the subsequent processing and culling of data (see Metrics section below)

What design do we have?
-----------------------

For (1) we have an extant design that allows that association of machine-readable data (specifically telemetry stored in the EFD) to be associated with an exposure using the start and end time of the exposure (see `DMTN-082 <http://dmtn-082.lsst.io>`__)

For (2) there  is a discussion in\ `LTS-836 <http://ls.st/lts-836>`__ about how the observing scripts are processed by the observing queue.  [1/21: Needs confirmation that there is a design resulting from that discussion.]


For (3) the design is that the scheduler adds an observation to the queue specifying campaign parameters (eg, survey field name); these parameters are forwarded to the Camera Control System (CCS) which ensures that they are written to the file header. [see discussion at `dmtn-058 <https://dmtn-058.lsst.io>`__]

For (4) the logging working group produced a use-case for this [`LSE-490 <https://docushare.lsst.org/docushare/dsweb/Get/LSE-490/lse490_ElectronicLoggingSystemReport_rel1_20200925.pdf>`__], but it remains to be seen that this covers all situations where post-facto information is uncovered.

For (5) the design of the pipeline tasks keeps input run lists.

What data paths do we have?
---------------------------

For (1) the exposure table in the Consolidated Database (which contains the information that we conceptually think of as the ‘FITS header”) can be populated by using the stream-processed EFD tables in the Consolidated Database. The list of exposures used to construct this table can either be extracted from EFD events or it can be supplied during file ingest.

For (2) information about the script should be recordable in the EFD as it is available to DDS.

For (3) the information should be retrievable from the Butler registry, the EFD, and the exposure table.

For (4) exposure (human-entered) notes will be entered into a separate database [check this], the access to this database and potential ways to status/flag exposures needs to be revisited.

For (5) the input run lists are kept [where?]

What is the state of implementation?
------------------------------------

For (1) we are capturing the relevant telemetry in the EFD. We are not currently constructing the exposure table but this work is planned. [1/21: note connection to ongoing discussions about how to build the exposure/visit tables envisioned in the DPDD, currently being discussed in the SDM-standardization meetings]

For (2) currently only the path to the observing script is being recorded and not generally retrievable.

For (3) this information is not currently in the header. This work is planned.

For (4) this information is being designed, the write interface is currently being implemented on both the backend (DB) and a front-end (LOVE). Read interface???

For (5) run lists have been used for some time.

Recommendations
---------------

The general approaches and notional designs seem reasonable, though there remain significant holes in the extant functionality. Following are the recommendations:

-  [REC-EXP-1]



Telemetry-level provenance
==========================

By “telemetry-level provenance” we mean associating observatory telemetry with properties of their originating systems (such as the name and version of a CSC) and allow their association with key observatory events (such as maintenance procedures).

What do we want?
----------------

Fundamentally we need to capture the instantaneous state of the system and what conditions it is operating in for situational awareness and to ensure appropriate and responsible scientific rigorousness in data recording.
This includes a complete picture of the states of all the subsystems, and the surrounding observatory environment (including the aspects of the visible sky, e.g. transmission, brightness).
For telemetry values we would like to capture their origin, including properties (including software versions) of the CSCs (Commandable Software Components) that produced them. 

A separate record of maintenance and other changes in the hardware is made in a separate MMS (maintenance management system) database and personnel notes and observations are recorded in the observatory-wide logging system.

What design do we have?
-----------------------

The EFD is designed to capture any time-series information accompanying
telemetry values in a DDS topic. (`SQR-29 <http://sqr-029.lsst.io>`__)
The Large File Annex (LFA) stores and archives larger (array) data
files, such as all-sky camera images, webcam images (or movies), and
input maps for the scheduler to be used in real-time or offline
analyses.

Both the OWL and MMS are still under design.

What data paths do we have?
---------------------------

Desired provenance data can be inserted and acquired via the SAL XML
interface, eg. https://ts-xml.lsst.io/sal_interfaces/ATCamera.html#softwareversions
The LFA is currently implemented as a local S3 service on the summit and will be synced to the USDF regularly (VERIFY THIS), so can be easily added to as more systems are brought online.
Observatory human logging including operator comments (for both timely and offline annotation of images and miscellaneous temporal events), in a dedicated database, and shall be accessible via RSP.
Similarly any hardware changes across the observatory are recorded in a separate Maintenance Management System (MMS) database which is still under construction.
At the very least, if there is not an on-demand API data stream access, this system shall have regularly exported YAML (or equivalent) data files with all relevant hardware change/alarm/status events (and attributable details).
This will be available for ingestion and assimilation by users at the RSP.

What is the state of implementation?
------------------------------------

The software architecture is mature and in production.
However only a minority of CSCs publish all this information at this time.
More CSCs are being added all the time as missing relevant aspects are being identified (e.g. seismic sensors, GIS, HVAC) and will likely continue into operations.

Data will be accessed by the users by multiple use-cases.

-  operators/engineers via LOVE (operator’s console), EUIs (engineer’s consoles), RSP notebook aspect, or Chronograf visualization interfaces.
-  scientists/external users via notebook aspect database access or butler if the associated telemetry is identified as critical information to an exposure
-  LFA data shall be accessible via RSP either through direct raw data access or via a specific butler or butler-like ingestion method if deemed necessary for the project and/or community.

Areas of concern focus on identifying all relevant aspects of the system and recording them in the EFD. A standard way (salobj) of implementing CSCs has improved the process somewhat. The MMS and OWL implementation is uncertain whether they will meet all provenance needs.

.. image:: Pictures/10000201000005000000027EE5DCFF60E7C8F918.png
   :width: 6.5in
   :height: 3.2398in

Recommendations
---------------

-  [REQ-TEL-001]   



Software-level provenance
=========================


We define software-level provenance as the type of provenance information that:

1. Records the names and versions of the software that were participants in the system state of interest; for example “what were the camera readout parameters at the time this observation was taken”
2. (Ideally, when possible) makes these available in a way that would allow the system to be reconfigured back to that state.

Relevant systems include the configuration of summit systems at the time of data taking, or the versions of science pipeline packages at the time of data processing.

In reality, this means we need several systems to capture provenance of the various contexts relevant to software:

-  Software versions
-  Container versions
-  Software configuration
-  System configuration: e.g. voltages
-  Schema evolution

All of these contexts are relevant to multiple subsystems in the project. For the purposes of this document we refer to two specific subsystems: Data Management (DM) and Telescope and Site (T&S).
T&S is intended to mean all software related to the physical instruments, the building, any test stands or auxiliary instrumentation, and infrastructure systems like HVAC regardless of actual WBS breakdown.

What do we want?
----------------

We want to know (and be able to reproduce) what telescope and instrument software versions were deployed when data taking occurred.
For example these could be wavefront sensing configurations, camera readout parameters, pointing models etc. 

What design do we have?
-----------------------

-  Software version

   -  DM -- All software is versioned via git and SHA1 hashes. There is also a release versioning system. The release versioning is not semantic.

   -  T&S -- All software is versioned via git and SHA1 hashes. Release versions of the SAL code is handled by ?? [KSK]

-  Container versions

   -  DM -- Container images are produced and uploaded to a container repository like DockerHub. As with software the containers have an associated unique hash so they can be identified. The Dockerfile used to produce the images is versioned via git, however, I’m unsure if there is a mechanism for matching up a given image with a git revision of a Dockerfile.

   -  T&S -- As in DM container images are uploaded to a container repository and images have a unique hash for identification. KSK] I’m unsure how Dockerfiles are versioned in T&S. I assume it’s git.

-  Software Configuration

   -  DM -- In DM, software configuration for the algorithms is handled by the configuration system of the pipeline tasks. This is discussed more in the PipelineTask provenance section. Configuration of many of the DM services is handled via a GitOps workflow mediated by the ArgoCD tool.

   -  T&S -- [KSK] I do not know how software configuration is handled in T&S

-  System Configuration

   -  DM -- By definition, there is little in the way of system configuration in DM. The computing hardware will have some configuration. Perhaps that will be captured in the processing metadata

   -  T&S -- [KSK] I do not know how system configuration is handled in T&S. E.g. how are the voltages on the camera set? Of course much of this information is captured in the telemetry stream but how configuration is versioned, stored and applied is not clear to me

-  Schema evolution

   -  DM -- Schemas for the data products are stored in git and are versioned like other software. Of course, versioning of the schema does not capture when it was applied to the running database instances, so there could be room for a recommendation with that. Schema for services are versioned by the avro/kafka schema migration machinery.

   -  T&S -- The message schemas are tightly controlled via XML documents that are versioned in git. They have a very strict release process that rolls out changes in the schema to running CSCs as a synchronized event (I believe). [KSK] I’m unaware of other schemas in the T&S sub-system that require provenance.

What data paths do we have?
---------------------------

In some cases data paths are either not determined or cannot be guaranteed to be correct.
For example, a concern would be that firmware is being updated “by hand” and that there is no way to “put a system back” to a specific state (eg during a particular testing run) because
it is not known what firmware was loaded at the time.

What is the state of implementation?
------------------------------------

Discussions are underway between T&S and SIT-COM on scoping this problem.

Recommendations
---------------

Questions related to recommendations:

-  Do we have a mechanism for matching up Dockerfile revisions with
   docker images? Do we care? This is hierarchical because all
   Dockerfiles start with a base image, so we’d want to match the
   Dockerfile for the base image as well.
-  How does T&S do software versioning?
-  Are there kinds of software configuration other than algorithmic and
   services?
-  How does T&S do software configuration?
-  How does T&S do system configuration?

Actual recommendations:

-  [REC-SW-1] Software provenance should be captured in association with
   “jobs” - units of execution. It should be possible to obtain software
   provenance data starting with a job ID.
-  [REC-SW-2] Versions of Rubin-controlled software should be captured
   using a small number of common mechanisms that each cover as broad a
   set of packages as possible.
-  [REC-SW-3] Software provenance should be captured in a
   machine-readable form which can be applied programmatically to
   reconstruct in an executable state the software configuration of a
   past execution. This does not imply an open-ended requirement to
   reconstruct old configurations when underlying O/S changes and the
   like have made that realistically infeasible.
-  [REC-SW-4] For as long as ‘eups’ is the principal tool for
   configuration management for the Science Pipelines software, software
   provenance for that software should be maintained in an
   ‘eups’-compatible form.
-  [REC-SW-5] Software provenance support should include mechanisms for
   capturing the versions of underlying non-Rubin software, including
   the operating system, standard libraries, and other tools which are
   needed “below” the Rubin software configuration management system.
   The use of community-standard mechanisms for this is strongly
   encouraged.
-  Containers should be used wherever is reasonable. This particularly
   applies to services or CSCs. They are easy to version and aid in
   repeatability by containing more than just version information such
   as configuration and system state.
-  A system for recording when versioned schema are applied to the
   running systems should be available
-  A census of services in both DM and T&S should be conducted. For any
   services that are not running in the, now standard, service
   infrastructure: i.e. kubernetes and argoCD for deployment, a specific
   explanation of why a migration is not necessary should be provided

PipelineTask-level provenance
=============================

PipelineTask-level provenance is the lowest level of provenance available through the LSST Science Pipelines without adding dedicated provenance-recording logic directly into the algorithmic code.
The middleware provides no means of automated detection and recording of provenance at the Task level, for Tasks contained in PipelineTasks.
Nor do we record sub-Task-level pixel arithmetic operations, such as recording that a dataset is the result of a subtraction between two antecedent exposures and then a multiplication, so for example data manipulated in a Science Platform notebook will not provide provenance as the design currently stands.

PipelineTask-level provenance, by contrast, is available “for free” from the design of the Gen3 middleware system, where inputs and outputs are mediated by the Butler and all PipelineTasks are executed by core Gen3 code.

The natural level of provenance-recording by this system will associate datasets, identified by DataId and type, and the collection in which they occur, with the PipelineTasks that produced them, identified by name and class, and the as-executed values of their configuration
objects.

Note that the system is only capable of recording that a given input was presented to a PipelineTask, not that the data in that input was actually used in the generation of the final result (e.g., it might fail a quality cut and not in fact be included in a coadd).

Additionally, it appears *(needs confirmation)*\ that as-executed lists of package versions, and physical dataset locators *(URIs?)* are recorded by the command-line activator (pipetask in ctrl_mpexec).

What do we want?
----------------

Provenance capture
^^^^^^^^^^^^^^^^^^

For a given output dataset of a PipelineTask we want to capture:

1. The specific versions of the PipelineTask stack that were run to create it;
2. The computing environment within which it was run;
3. The specific configuration (pex_config) that was applied, after the “stacking up” of all defaults and overrides;
4. The input datasets presented to the PipelineTask that generated the output, ideally named in both site-independent and physical forms;
5. Any QA metrics that were generated “in situ” as part of the calculational work of the PipelineTask;
6. Logs and/or other outputs to indicate success/failure performance, etc.

Provenance utilization
^^^^^^^^^^^^^^^^^^^^^^

We want to be able to perform queries against the recorded provenance, such as “tell me which raws or which calexps contributed to this coadd” from the Butler (see figure for a visual aid).

The above capture and query capability is reflected in DMS-MWBT-REQ-0094 & DMS-MWBT-REQ-0095 (`LDM-556 <http://ldm-556.lsst.io>`__) and ultimately flows down via LSE-61 from LSE-30 (OSS-REQ-0122) which requires that sufficient provenance is recorded that data products can be reproduced.

We would like to have both code and command-line support for the operation “re-run, as exactly as possible, the processing that was used to generate dataset X”, based on stored provenance.
This would, for instance, use the frozen “as-executed” configuration values as a 100% override to any default configuration values in the code used for the re-run.
This re-run capability is needed for validation as well as for use in “virtual data product re-creation” services.
It will also be needed by Notebook Aspect users.

.. image:: Pictures/100002010000050000000290F5389AC0A7C18C30.png
   :width: 6.5in
   :height: 3.3311in


Additionally we would like a provenance web service to allow Science Platform users to perform these queries, such as the IVOA provenance ProvDAL service.

We are not aware of any work that has been done to date on mapping the PipelineTask provenance to common community three-term ontologies for provenance such as the W3C or IVOA provenance models. However, the information content seems likely to have a fairly natural mapping.

What design do we have?
-----------------------

`LDM-152 <http://ldm-152.lsst.io>`__ specifies that the configuration and inputs to PipelineTasks are preserved.

What data paths do we have?
---------------------------

The Science Pipelines executor currently records software versions and configuration in the Butler.
In the design, the executor stores the quantum graph in the Butler in a form that would allow an API to service the example queries above.

What is the state of implementation?
------------------------------------

(1) and (2) are stored and queryable by the Butler API while (3) is not yet implemented but is planned.

VO access to this information via ProvDAL is not planned in construction.

Recommendations (Below is an attempt to “finish off” this section)

-  Item (3), the configuration used in a re-run should be trace-able (like the software version).
-  At the very least the quantum graph should be stored as it expresses the relationship between the inputs and outputs and tasks (within the context of a given software stack and configuration).
-  The tools to query such provenance can be left to the user (presumably DM/OPS staff would build these as needed/warranted).

Workflow-level provenance
=========================

Note that in our architecture, some of the provenance use cases that are typically the domain of the workflow system (such as software version provenance) are handled by Task-Level provenance.\ *(Verify with Middleware - this would make it a requirement on any activator)*
This is also an effect of the design where there are elements of the Science Pipelines (specifically pipe_base) that is “upstream” of the workflow system, as it generates the quantum graph submitted to the workflow.

There is metadata associated with workflow (such as log messages generated during a particular run and the configuration of the execution node itself), but there is no provenance tree associated with them.

Need a discussion of resource-usage information here as well - since this has site-dependent aspects.

`LSE-30 <http://ls.st/lse-30>`__ does require operating system and
hardware provenance to be recorded. This could be done at workflow-level provenance, but given the lack of requirement at this level it might be simpler to just add this information to task-level provenance (where the OS is already recorded but not the version).

Recommendations
---------------

File-level provenance
=====================

We define file-level provenance as the inputs that contributed to the production of that data, including other files and software.
There are various ways of represent these, eg. a graph of predecessor data.
By tracing a provenance chain one can then reconstruct the relationship of products to upstream or downstream products and processes.

An alternative means to express provenance would take the form that associates a collection of inputs and outputs, along with a record of a broader pipeline task and configuration.
The granularity of such provenance is not amenable to answering questions about how a product
was used without *a priori*\ knowledge of the pipeline processing, but can be much faster for certain search operations. 

Both the above cases can be thought of as an extrapolation of PipelineTask- and Workflow-level provenance to the file level.
The two cases are not mutually exclusive (ie. they could both be persisted).
In fact the methods for exploiting the information can be left to the users, so long as the relational information is systematically stored.

What do we want?
----------------

There are two relevant requirements in `LDM-556 <http://ldm-556.lsst.io>`__:

1. Persisting provenance information with the raw data IDs that contributed to a dataset into the final export data format (be it FITS or alternative) (DMS-MWBT-REQ-0093)
2. Same but with the immediate parents (eg in the diagram above, the parents of the Coadd pictured are the CalExps/PVIs) (DMS-MWBT-REQ-0093)

What design do we have?
-----------------------

There is no current design for implementing this. Three options would be:

- “Burning it” into the file on write (on Butler Put)
- Packaging it with the file on read/export (by the service publishing the file)
- Saving relational information in the Butler registry and leaving the methodology for its retrieval/use/exploitation to the user.

An alternative to this approach would be to fulfil the spirit of the requirement by burning into the file a service call (eg. DataLink) that supplies the required provenance information.
Metadata such as the run collection, dataId, and dataset type are not (currently) stored in persisted formats.

The filename should not be relied to for provenance lookup since it may be changed by the user and furthermore the filenames alone cannot be relied on because they are not unique to a specific processing attempt of a given product.

Finally, it is often NOT desirable to express all parent files that ever led to the creation of a data product as part of that product.
For example, recording every flat field that was used in the generation of a CalExp that in turn was used as part of a COADD image would be wasteful.
The record of such relations is better stored in a database (eg. Butler registry) where it can be queried than accumulated/persisted in the header of each output image.
The unanswered question is whether there are cases where such file level provenance information should be saved in an image header.

What data paths do we have?
---------------------------

The information is known as part of the Task-Level provenance above.

What is the state of implementation?
------------------------------------

Not currently implemented.

We are concerned that data processing and imminently data-taking is underway prior to a system to record this provenance information is in existence. 

Recommendations
---------------

-  [REC-FIL-1] Serialised exported data products (FITS files in the requirements) should include file metadata (eg. FITS header) that allows someone in possession of the file to come to our services and query for additional provenance information for that artifact (eg pipeline-task level provenance).

- [REC-FIL-2] A study should be made of the possibility of embedding a DataLink or other service pointer in the FITS header in lieu of representing the provenance graph in the file

- [REC-FIL-3] Irrespective of ongoing design discussions, every attempt should be made to capture information that could later be used to populate a provenance service. 


Source-level provenance
=======================

By source-level provenance we mean astronomical sources in catalogs (sources, objects, etc). For simplicity we use "Source ID" in this section to mean the appropriate identifier of any source-like product (DIAsource, DIAObject, Object, etc)

What do we want?
----------------

We agree with `DMTN-085 <http://dmtn-085.lsst.io>`__ (report of the QA working group) that there is no strong requirement for pixel level per-source/object provenance beyond an association with the dataset from which the source measurement was derived since  we are no longer using the multifit approach (and its multiple source simultaneous source model fitting approach).

However, there are per-source metadata that need to be propagated to the final data release product.
The two that we have identified are flags and footprints.

Flags include boolean information about the source detection quality, e.g., were there saturated pixels in the detection.
Flags can also be used to capture processing information such as which objects were used for astrometric calibration, photometric calibration, PSF modeling, and whether a source is an injected fake. 

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

Though the footprints are computed as part of processing, and are persisted as intermediate products, there is no implementation for providing them to end users (they are available directly through the butler in gen 3).

Heavy footprints are not in the sizing model or the DPDD. *fact check*

Recommendations
---------------

- [REC-SRC-001] Perform a census of produced and planned flags to ensure that 64 bits for sources and 128 bits for objects is sufficient within a generous margin of error. This activity should also be carried out for DIASources and DIAObjects source IDs.

- [REC-SRC-002] We are concerned that merely encoding a 4-bit data release provenance in a source does not scale to commissioning needs and the project should decide whether it is acceptable for additional information beyond the source ID to be required to fully associate a source with a specific image.

- [REC-SRC-003] More generally, a study should be conducted on whether 64 bit source IDs are sufficient

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

The Butler team is planning for the low level executor for pipeline tasks to generate a unique identifier for a pipeline execution run, which effectively can be used as the "job ID" initially envisaged.

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

Storing metrics in the Butler as ad-hoc datasets signicantly limits the usability and utility of these metrics. If the metrics were supported  as a native structured Butler dataset, then we would be able to

1. Query the Butler for what metrics are available (metrics discovery)
2. Have the ability to filter other Butler queries on the basis of metric measurements
3. Significantly increase the robustness of metric transport to Squash by associating the lsst.verify metrics specification with the Butler 

We understand such development is not planned in construction. 
   
*[maybe a diagram/example]* 

Recommendations
---------------

-  [REC-MET-001] For metrics that can be associated with a Butler dataId, the metrics should be persisted using the Data Butler as the source of truth. The dataId associated with the metric should use the full granularity
-  [REC-MET-002] Any system that uses Butler data to derive metrics should persist them in the Butler provided that the metrics are associable with a Data ID
-  [REC-MET-003] When lsst.verify.Job objects are exported, the exported object should included the needed information (run collection and dataId) to associate with the source of truth metric persisted with Data Butler
-  [REC-MET-004] A plan should be developed for persisting metrics that are not directly associated with non-Butler persisted metrics.
- [REC-MET-005] Even if effort from implementation is not available in construction, we should develop a conceptual design for structured, semantically rich storage of metrics in the Butler


Log Provenance
==============

What do we want?
----------------

Logs, i.e., machine-generated output from software and systems involved
in data taking are sometimes necessary in order to understand unexpected
behaviour. Log provenance shares most provenance requirement with
metrics data, except for being a

What data paths do we have?
---------------------------


What is the state of implementation?
------------------------------------


Recommendations
---------------

-  [REC-LOG-1]

Additional notes:
=================

-  Is it possible to have a more fine-grained approach to provenance,
   where some extra intermediates and parent files (heavy footprints,
   PVIs etc) are kept for the first data release where we anticipate
   that the trust phase will unbalance the usual space-time trade-off,
   and also observe what the usage of these products are? This has a
   different sizing impact that assuming that if we keep something we
   need the sizing model to support it forever

-  Could we have a “gold master verification patch” where we keep
   everything in order to allow people to “check our work” at whatever
   level they wish without blowing up the sizing model or figuring out
   how to systematically store those products/provenance over all the
   lifetime of the project?

-  Amanda to go through EPO use cases:

   -  “FITS header” elements needed to produce “pretty pictures” for
         public audiences

   -  webcam images to display on a public-facing “observatory status
         dashboard” webpage.

- We should require a glpbal provenance key for all data curating provenance associating all curated artifacst with a time and if possible a data association. This is to allow collation of provenance curated by heterogenous systems. Original phrasing follows:  [REC-MET-004] As suggested by the QA Strategy Working Group (`DMTN-085 <https://dmtn-085.lsst.io/>`__), collections of related metric values should be stored in a format that can be efficiently queried and joined with survey metadata (e.g., telemetry, exposure id, survey property maps). This data store should be associated with the Data Butler.


.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
