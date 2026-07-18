# EDMO-Enrollment-Intelligence-Integration
Data model first, then REST → Trigger → LWC → Tests, each layer built on the one below it.
REST endpoint (EnrollmentScoreRestService) resolves all Contact emails in one bulk query, uses Database.insert(..., false) for partial success, and returns a per-record RecordResult so one bad row never fails the batch.
Trigger is thin (EnrollmentScoreTrigger); all logic lives in EnrollmentScoreTriggerHandler, which reduces a batch to one "latest Scored_At__c wins" score per Contact before a single bulk update.
LWC (enrollmentScoreCard) uses @wire for reactive fetch, returns null (not an error) when no score exists, and shows a color-coded Hot/Warm/Cold badge.
Security: with sharing + WITH SECURITY_ENFORCED + explicit isAccessible()/isCreateable()/isUpdateable() checks throughout, so the integration respects CRUD/FLS/sharing rather than running as System.
Tests: one class, 11 methods, asserting actual persisted values (not just isSuccess()), covering all three priority tiers including boundary values.
Trade-off: WITH SECURITY_ENFORCED fails the whole request rather than degrading per-row if the integration user lacks FLS — acceptable given a dedicated permission set is expected to be provisioned.
