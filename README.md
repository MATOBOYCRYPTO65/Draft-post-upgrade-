# Draft-post-upgrade-
Proxy Upgrade Scenario
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/schemas/stva-proxy-upgrade-policy.schema.json",
  "title": "STVA Proxy Upgrade Validation Policy",
  "description": "Policy for validating STVA labels and execution summaries for proxy upgrade transactions. Requires that intent be distinguished from implementation safety claims.",
  "type": "object",
  "additionalProperties": false,
  "required": [
    "transaction",
    "verification",
    "assumptions",
    "executionOutcome",
    "status"
  ],
  "properties": {
    "transaction": {
      "type": "object",
      "additionalProperties": false,
      "required": [
        "kind",
        "target",
        "parameterChanges"
      ],
      "properties": {
        "kind": {
          "type": "string",
          "const": "proxy_upgrade"
        },
        "target": {
          "type": "string",
          "pattern": "^0x[a-fA-F0-9]+$",
          "description": "Proxy or Safe contract address"
        },
        "parameterChanges": {
          "type": "array",
          "minItems": 1,
          "items": {
            "type": "object",
            "additionalProperties": false,
            "required": [
              "slot",
              "newValue"
            ],
            "properties": {
              "slot": {
                "type": "string",
                "enum": [
                  "implementation",
                  "admin",
                  "owner"
                ]
              },
              "newValue": {
                "type": "string",
                "pattern": "^0x[a-fA-F0-9]+$"
              }
            }
          }
        }
      }
    },
    "verification": {
      "type": "object",
      "additionalProperties": false,
      "required": [
        "signingIntent",
        "evidenceScope",
        "verified"
      ],
      "properties": {
        "signingIntent": {
          "type": "string",
          "minLength": 1,
          "description": "Intent authorized by signer, e.g. 'Upgrade Safe implementation to 0xNewImplementation'"
        },
        "evidenceScope": {
          "type": "string",
          "enum": [
            "transaction_intent_only",
            "state_change_only",
            "transaction_intent_and_state_change"
          ]
        },
        "verified": {
          "type": "boolean",
          "const": true
        }
      }
    },
    "assumptions": {
      "type": "array",
      "description": "Assumptions evaluated during simulation; never converted into security guarantees.",
      "minItems": 1,
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": [
          "statement",
          "status"
        ],
        "properties": {
          "statement": {
            "type": "string",
            "minLength": 1
          },
          "status": {
            "type": "string",
            "enum": [
              "assumed_true",
              "not_evaluated",
              "requires_external_verification"
            ]
          }
        }
      }
    },
    "executionOutcome": {
      "type": "object",
      "additionalProperties": false,
      "required": [
        "enforcedChanges",
        "assumptionBoundaries",
        "safetyClaim"
      ],
      "properties": {
        "enforcedChanges": {
          "type": "array",
          "minItems": 1,
          "items": {
            "type": "string",
            "enum": [
              "implementation_slot_changed",
              "admin_slot_changed",
              "owner_slot_changed"
            ]
          }
        },
        "assumptionBoundaries": {
          "type": "array",
          "minItems": 1,
          "items": {
            "type": "string",
            "minLength": 1
          }
        },
        "safetyClaim": {
          "type": "string",
          "enum": [
            "assumed_not_verified",
            "not_claimed",
            "requires_independent_audit"
          ]
        },
        "newImplementationSafetyVerified": {
          "type": "boolean",
          "const": false
        }
      }
    },
    "status": {
      "type": "string",
      "enum": [
        "pass",
        "fail"
      ]
    },
    "warnings": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "disallowedClaimMatches": {
      "type": "array",
      "description": "Optional list of claims explicitly rejected by this policy.",
      "items": {
        "type": "string",
        "enum": [
          "verified and safe",
          "secure new implementation",
          "This transaction is verified and safe"
        ]
      }
    }
  },
  "allOf": [
    {
      "if": {
        "properties": {
          "status": {
            "const": "pass"
          }
        }
      },
      "then": {
        "properties": {
          "executionOutcome": {
            "properties": {
              "safetyClaim": {
                "enum": [
                  "assumed_not_verified",
                  "not_claimed",
                  "requires_independent_audit"
                ]
              }
            }
          }
        }
      }
    }
  ]
}
{
  "transaction": {
    "kind": "proxy_upgrade",
    "target": "0xSafeProxy",
    "parameterChanges": [
      {
        "slot": "implementation",
        "newValue": "0xNewImplementation"
      }
    ]
  },
  "verification": {
    "signingIntent": "Upgrade the Safe implementation to 0xNewImplementation",
    "evidenceScope": "transaction_intent_and_state_change",
    "verified": true
  },
  "assumptions": [
    {
      "statement": "The new implementation is not malicious",
      "status": "assumed_true"
    },
    {
      "statement": "No additional storage collisions occur",
      "status": "assumed_true"
    }
  ],
  "executionOutcome": {
    "enforcedChanges": [
      "implementation_slot_changed"
    ],
    "assumptionBoundaries": [
      "Only the implementation slot change is enforced by this transaction",
      "All safety properties of the new implementation are assumptions"
    ],
    "safetyClaim": "assumed_not_verified",
    "newImplementationSafetyVerified": false
  },
  "status": "pass",
  "warnings": [],
  "disallowedClaimMatches": []
}
{
  "transaction": {
    "kind": "proxy_upgrade",
    "target": "0xSafeProxy",
    "parameterChanges": [
      {
        "slot": "implementation",
        "newValue": "0xNewImplementation"
      }
    ]
  },
  "verification": {
    "signingIntent": "Secure new implementation",
    "evidenceScope": "transaction_intent_only",
    "verified": true
  },
  "assumptions": [],
  "executionOutcome": {
    "enforcedChanges": [
      "implementation_slot_changed"
    ],
    "assumptionBoundaries": [],
    "safetyClaim": "not_claimed",
    "newImplementationSafetyVerified": false
  },
  "status": "fail",
  "warnings": [
    "Claim overstates verification; new implementation safety is not established."
  ]
}
