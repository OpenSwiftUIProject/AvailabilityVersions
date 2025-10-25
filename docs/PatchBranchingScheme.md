# Patch Branching Scheme for AvailabilityVersions Fork

This document describes the branching and release scheme for this community-maintained fork of AvailabilityVersions.

## Overview

Since Apple has stopped updating the official AvailabilityVersions project after version 155 (iOS 17.3), this fork maintains compatibility with newer OS releases by reconstructing missing availability data from dyld binaries.

## Branch Organization

### Upstream Branches

Upstream branches track Apple's official releases:

- **Branch**: `rel/AvailabilityVersions-{VERSION}`
  - Example: `rel/AvailabilityVersions-155`
- **Tag**: `AvailabilityVersions-{VERSION}`
  - Example: `AvailabilityVersions-155`

These branches are **read-only** and track the last known official Apple release for each version.

### Patch Branches

Patch branches contain our community updates:

- **Branch**: `rel/patch/AvailabilityVersions-{VERSION}`
  - Example: `rel/patch/AvailabilityVersions-155`
- **Tag**: `AvailabilityVersions-{VERSION}-patch`
  - Example: `AvailabilityVersions-155-patch`
  