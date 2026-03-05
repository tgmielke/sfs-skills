---
name: sf-fs-datacapture
description: >
  Creates Field Service data capture form flows from images or PDFs of existing
  paper/digital forms. Generates mobile-optimized DataCaptureFlow XML with
  Field Service DC components, voice-to-form (IsLlmTargetable), offline support,
  and deploys to Salesforce orgs. Use when building Field Service data capture
  forms, inspection checklists, safety forms, or converting paper forms to
  digital flows for field technicians.
---

# sf-fs-datacapture: Field Service Data Capture Form Builder

Build mobile-optimized data capture flows for Salesforce Field Service from images, PDFs, or descriptions of existing forms. Generates deployable Flow metadata XML with voice-to-form support.

## Quick Reference

| Setting | Value |
|---------|-------|
| processType | `DataCaptureFlow` |
| environments | `Offline` |
| Voice-to-Form | `customProperties > IsLlmTargetable > {"value":"true"}` |
| API Version | 65.0+ |
| Naming | `DCF_[Context]_[FormName]` |
| Input vars | `parentRecordId`, `parentObjectType`, `recordId` |
| Deploy path | `force-app/main/default/flows/` |

---

## Phase 1: Form Analysis (Image/PDF Intake)

When the user provides an image or PDF of a form:

1. **Examine every field** — labels, input types, groupings, sections
2. **Classify each field** using the DC component mapping below
3. **Identify sections** — group related fields into logical screens (5-7 fields max)
4. **Note conditional logic** — fields that depend on toggle/answer states
5. **Check for signatures, photos** — these use specialized DC components

### Field Type Mapping (Document → DC Component)

Data capture flows use **Field Service DC components**, NOT standard flow field types.

| Document Field | extensionName | fieldType | Notes |
|----------------|---------------|-----------|-------|
| Text box (any length) | `runtime_service_fieldservice:dcTextInput` | `ComponentInstance` | Handles both short and long text |
| Dropdown / picklist | `runtime_service_fieldservice:dcPicklist` | `ComponentChoice` | Uses `choiceReferences` |
| Checkbox group (multi) | `runtime_service_fieldservice:dcCbGroup` | `ComponentMultiChoice` | Uses `choiceReferences` |
| Toggle / Yes-No | `runtime_service_fieldservice:dcToggle` | `ComponentInstance` | Boolean toggle, ref via `.isActive` |
| Photo / camera | `runtime_service_fieldservice:dcUpImage` | `ComponentInstance` | Set `useCameraOnly` input param |
| Signature | `runtime_service_fieldservice:dcSignature` | `ComponentInstance` | Pass `parentRecordId` input param |
| Section header | _(none)_ | `DisplayText` | HTML-formatted `fieldText` |
| Instructions / read-only | _(none)_ | `DisplayText` | Informational text |

### DC Text Input Pattern (most common field)

```xml
<fields>
    <name>{{Field_Name}}</name>
    <extensionName>runtime_service_fieldservice:dcTextInput</extensionName>
    <fieldType>ComponentInstance</fieldType>
    <inputParameters>
        <name>label</name>
        <value>
            <stringValue>{{Field Label}}</stringValue>
        </value>
    </inputParameters>
    <inputsOnNextNavToAssocScrn>UseStoredValues</inputsOnNextNavToAssocScrn>
    <isRequired>{{true|false}}</isRequired>
    <storeOutputAutomatically>true</storeOutputAutomatically>
    <styleProperties>
        <verticalAlignment>
            <stringValue>top</stringValue>
        </verticalAlignment>
        <width>
            <stringValue>12</stringValue>
        </width>
    </styleProperties>
</fields>
```

For required fields, also add the `required` input parameter:
```xml
<inputParameters>
    <name>required</name>
    <value>
        <booleanValue>true</booleanValue>
    </value>
</inputParameters>
```

### DC Picklist Pattern (single-select dropdown)

```xml
<fields>
    <name>{{Field_Name}}</name>
    <choiceReferences>{{Choice_1}}</choiceReferences>
    <choiceReferences>{{Choice_2}}</choiceReferences>
    <extensionName>runtime_service_fieldservice:dcPicklist</extensionName>
    <fieldText>{{Field Label}}</fieldText>
    <fieldType>ComponentChoice</fieldType>
    <inputsOnNextNavToAssocScrn>UseStoredValues</inputsOnNextNavToAssocScrn>
    <isRequired>{{true|false}}</isRequired>
    <storeOutputAutomatically>true</storeOutputAutomatically>
    <styleProperties>
        <verticalAlignment><stringValue>top</stringValue></verticalAlignment>
        <width><stringValue>12</stringValue></width>
    </styleProperties>
</fields>
```

### DC Checkbox Group Pattern (multi-select)

```xml
<fields>
    <name>{{Field_Name}}</name>
    <choiceReferences>{{Choice_1}}</choiceReferences>
    <choiceReferences>{{Choice_2}}</choiceReferences>
    <extensionName>runtime_service_fieldservice:dcCbGroup</extensionName>
    <fieldText>{{Field Label}} (Select All That Apply)</fieldText>
    <fieldType>ComponentMultiChoice</fieldType>
    <inputsOnNextNavToAssocScrn>UseStoredValues</inputsOnNextNavToAssocScrn>
    <isRequired>false</isRequired>
    <storeOutputAutomatically>true</storeOutputAutomatically>
    <styleProperties>
        <verticalAlignment><stringValue>top</stringValue></verticalAlignment>
        <width><stringValue>12</stringValue></width>
    </styleProperties>
</fields>
```

### DC Toggle Pattern (boolean with conditional visibility)

```xml
<fields>
    <name>{{Toggle_Name}}</name>
    <extensionName>runtime_service_fieldservice:dcToggle</extensionName>
    <fieldType>ComponentInstance</fieldType>
    <inputParameters>
        <name>label</name>
        <value><stringValue>{{Toggle Question}}</stringValue></value>
    </inputParameters>
    <inputsOnNextNavToAssocScrn>UseStoredValues</inputsOnNextNavToAssocScrn>
    <isRequired>{{true|false}}</isRequired>
    <storeOutputAutomatically>true</storeOutputAutomatically>
    <styleProperties>
        <verticalAlignment><stringValue>top</stringValue></verticalAlignment>
        <width><stringValue>12</stringValue></width>
    </styleProperties>
</fields>
```

Show/hide a field based on toggle state:
```xml
<visibilityRule>
    <conditionLogic>and</conditionLogic>
    <conditions>
        <leftValueReference>{{Toggle_Name}}.isActive</leftValueReference>
        <operator>EqualTo</operator>
        <rightValue>
            <booleanValue>true</booleanValue>
        </rightValue>
    </conditions>
</visibilityRule>
```

### DC Image Upload Pattern

```xml
<fields>
    <name>{{Upload_Name}}</name>
    <extensionName>runtime_service_fieldservice:dcUpImage</extensionName>
    <fieldType>ComponentInstance</fieldType>
    <inputParameters>
        <name>label</name>
        <value><stringValue>Upload Photos</stringValue></value>
    </inputParameters>
    <inputParameters>
        <name>useCameraOnly</name>
        <value><booleanValue>true</booleanValue></value>
    </inputParameters>
    <inputsOnNextNavToAssocScrn>UseStoredValues</inputsOnNextNavToAssocScrn>
    <isRequired>{{true|false}}</isRequired>
    <storeOutputAutomatically>true</storeOutputAutomatically>
    <styleProperties>
        <verticalAlignment><stringValue>top</stringValue></verticalAlignment>
        <width><stringValue>12</stringValue></width>
    </styleProperties>
</fields>
```

### DC Signature Pattern

```xml
<fields>
    <name>{{Signature_Name}}</name>
    <extensionName>runtime_service_fieldservice:dcSignature</extensionName>
    <fieldType>ComponentInstance</fieldType>
    <inputParameters>
        <name>label</name>
        <value><stringValue>Customer Signature</stringValue></value>
    </inputParameters>
    <inputParameters>
        <name>parentRecordId</name>
        <value><elementReference>parentRecordId</elementReference></value>
    </inputParameters>
    <inputsOnNextNavToAssocScrn>UseStoredValues</inputsOnNextNavToAssocScrn>
    <isRequired>false</isRequired>
    <storeOutputAutomatically>true</storeOutputAutomatically>
    <styleProperties>
        <verticalAlignment><stringValue>top</stringValue></verticalAlignment>
        <width><stringValue>12</stringValue></width>
    </styleProperties>
</fields>
```

---

## Phase 2: Flow Design

### Screen Organization for Mobile

1. **Max 5-7 fields per screen** — avoid scrolling on phones
2. **Group by section** — one section of the form = one screen
3. **Required fields at top** of each screen
4. **Short, clear labels** — technicians glance quickly
5. **Use `width: 6`** for side-by-side fields (e.g., Floor + Room Number)
6. **Use `width: 12`** for full-width fields (default)

### Screen Navigation

| Screen Position | allowBack | allowFinish | allowPause |
|-----------------|-----------|-------------|------------|
| First screen | `true` | `true` | `true` |
| Middle screens | `true` | `true` | `true` |
| Final screen | `true` | `true` | `true` |

### Naming Conventions

| Element | Pattern | Example |
|---------|---------|---------|
| Flow API name | `DCF_[Context]_[Name]` | `DCF_Safety_Hazard_Check` |
| Screen | `[Section]_Screen` | `Demarc_Location_Screen` |
| Text input | `[FieldName]` | `Circuit_ID`, `Building_Name` |
| Toggle | `[Toggle_Name]` | `Has_Blockers`, `Tests_Performed` |
| Choice | `[Category]_[Value]` or `Choice_[Value]` | `Cond_Good`, `Choice_High` |
| Display text | `[Purpose]` | `Display_WO_Context`, `Section_PPE` |

---

## Phase 3: XML Generation

### Required Flow Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Flow xmlns="http://soap.sforce.com/2006/04/metadata">
    <areMetricsLoggedToDataCloud>false</areMetricsLoggedToDataCloud>

    <!-- choices (alphabetical) -->

    <customProperties>
        <name>IsLlmTargetable</name>
        <value>
            <stringValue>{"value":"true"}</stringValue>
        </value>
    </customProperties>

    <description>{{DESCRIPTION}}</description>
    <environments>Offline</environments>
    <interviewLabel>{{LABEL}} {!$Flow.CurrentDateTime}</interviewLabel>
    <label>{{LABEL}}</label>

    <processMetadataValues>
        <name>BuilderType</name>
        <value><stringValue>LightningFlowBuilder</stringValue></value>
    </processMetadataValues>
    <processMetadataValues>
        <name>CanvasMode</name>
        <value><stringValue>AUTO_LAYOUT_CANVAS</stringValue></value>
    </processMetadataValues>
    <processMetadataValues>
        <name>OriginBuilderType</name>
        <value><stringValue>LightningFlowBuilder</stringValue></value>
    </processMetadataValues>

    <processType>DataCaptureFlow</processType>

    <recordLookups>
        <name>Get_Work_Order</name>
        <label>Get Work Order</label>
        <locationX>0</locationX>
        <locationY>0</locationY>
        <assignNullValuesIfNoRecordsFound>false</assignNullValuesIfNoRecordsFound>
        <connector>
            <targetReference>{{FIRST_SCREEN}}</targetReference>
        </connector>
        <filterLogic>and</filterLogic>
        <filters>
            <field>Id</field>
            <operator>EqualTo</operator>
            <value>
                <elementReference>parentRecordId</elementReference>
            </value>
        </filters>
        <getFirstRecordOnly>true</getFirstRecordOnly>
        <object>WorkOrder</object>
        <storeOutputAutomatically>true</storeOutputAutomatically>
    </recordLookups>

    <!-- screens -->

    <start>
        <locationX>0</locationX>
        <locationY>0</locationY>
        <connector>
            <targetReference>Get_Work_Order</targetReference>
        </connector>
    </start>

    <status>Draft</status>

    <variables>
        <name>parentObjectType</name>
        <dataType>String</dataType>
        <isCollection>false</isCollection>
        <isInput>true</isInput>
        <isOutput>false</isOutput>
    </variables>
    <variables>
        <name>parentRecordId</name>
        <dataType>String</dataType>
        <isCollection>false</isCollection>
        <isInput>true</isInput>
        <isOutput>false</isOutput>
    </variables>
    <variables>
        <name>recordId</name>
        <dataType>String</dataType>
        <isCollection>false</isCollection>
        <isInput>true</isInput>
        <isOutput>false</isOutput>
    </variables>
</Flow>
```

### Work Order Context Display (first field on first screen)

```xml
<fields>
    <name>Display_WO_Context</name>
    <fieldText>&lt;p&gt;&lt;b&gt;Work Order:&lt;/b&gt; {!Get_Work_Order.WorkOrderNumber}&lt;/p&gt;&lt;p&gt;&lt;b&gt;Subject:&lt;/b&gt; {!Get_Work_Order.Subject}&lt;/p&gt;</fieldText>
    <fieldType>DisplayText</fieldType>
    <styleProperties>
        <verticalAlignment><stringValue>top</stringValue></verticalAlignment>
        <width><stringValue>12</stringValue></width>
    </styleProperties>
</fields>
```

### XML Element Ordering (Alphabetical — MANDATORY)

```
areMetricsLoggedToDataCloud → choices → customProperties → description →
environments → interviewLabel → label → processMetadataValues →
processType → recordLookups → screens → start → status → variables
```

See [assets/data-capture-flow-template.xml](assets/data-capture-flow-template.xml) for the full base template.

---

## Phase 4: Deployment

```bash
mkdir -p force-app/main/default/flows
# Write: force-app/main/default/flows/DCF_[Name].flow-meta.xml
```

Use the **sf-deploy** skill:
1. `"Deploy flow with --dry-run"` — validate first
2. `"Proceed with actual deployment"` — deploys as Draft
3. Change `<status>Draft</status>` → `<status>Active</status>`, redeploy to activate

---

## Phase 5: Verification

- [ ] All screens render on Field Service mobile app
- [ ] Voice-to-form works (mic icon appears, dictation maps to fields)
- [ ] Toggle-based conditional fields show/hide correctly
- [ ] Field values persist when navigating between screens
- [ ] Photo capture opens device camera
- [ ] Signature pad accepts input
- [ ] Flow works offline (airplane mode test)
- [ ] Work Order context displays correctly on first screen

---

## Cross-Skill Integration

| From | To | When |
|------|-----|------|
| sf-metadata → here | Create custom objects/fields the form writes to |
| sf-flow → here | General flow patterns, subflow reuse |
| here → sf-deploy | Deploy form flow to target org |
| here → sf-data | Create test Work Orders / Service Appointments |

---

## Additional Resources

- [Data Capture Flow Template](assets/data-capture-flow-template.xml) — Base XML template
- [Field Type Mapping Reference](references/field-type-mapping.md) — All DC component patterns
- [Mobile UX Guide](references/mobile-ux-guide.md) — Field Service mobile best practices
- [Example: Safety Hazard Check](references/example-safety-hazard-check.xml) — Multi-select checklists, photo upload
- [Example: Customer Interaction](references/example-customer-interaction.xml) — Signature capture, satisfaction survey
- [Example: NAS Demarc Validation](references/example-nas-demarc-validation.xml) — Multi-screen, conditional visibility
