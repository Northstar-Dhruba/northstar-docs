# Story 2 - Explain Recommendation

## Status

**Ready for Development**

Story 2 delivers the next trust-building step in the Northstar Intelligence MVP: a user can understand why the recommendation produced for one analyzed asset was made.

This story preserves Northstar Architecture v2.0 and the completed Story 1 contracts. It adds explanation and supporting reasoning only. It does not introduce confidence, risk, watchlists, execution, or scanning.

---

## 1. User Story

> As an investor, I want Northstar to explain why a recommendation was made, so that I can decide whether I trust it.

---

## 2. Goal

Deliver the second complete product feature:

```text
Analyze Asset
  ↓
Recommendation
  ↓
Explanation
```

The user remains on the Analyze One Asset workspace and can read an explanation for the recommendation they have just received. No additional navigation or product workflow is required.

---

## 3. Business Purpose

A Recommendation Explanation is a user-facing, understandable account of the reasoning that supports one produced `Recommendation`.

It exists to answer: "Why did Northstar produce this recommendation for this asset at this point in time?"

An explanation supports a Recommendation. It does not determine, override, rank, validate, or modify a Recommendation. Recommendation meaning and BUY/HOLD/SELL policy remain owned by `Strategy`.

Supporting reasoning makes the relevant decision context understandable without introducing new decision inputs, recommendation rules, confidence, risk, or an AI summary beyond the explanation itself.

---

## 4. Relationship Review

The Story 2 concept flow is:

```text
AssetAnalysis
  ↓
Strategy
  ↓
Recommendation
  ↓
Recommendation Explanation
```

### AssetAnalysis

`AssetAnalysis` represents the interpreted market context for one listing at one point in time. It supplies the factual, summarized context used by `Strategy`.

It does not recommend an action or explain a recommendation to a user.

### Strategy

`Strategy` owns the recommendation policy. It evaluates an `AssetAnalysis` and determines the resulting BUY, HOLD, or SELL action.

It does not become a delivery concern and does not shift its decision policy into the Application, API, or Web layers.

### Recommendation

`Recommendation` is the Strategy-produced decision for one `AssetAnalysis` and Strategy identity at one point in time.

It remains the authoritative outcome. Story 2 must not add any behavior that recalculates or alters its action.

### Recommendation Explanation

A Recommendation Explanation expresses the supporting reasoning for one existing Recommendation. It is derived from the recommendation and its existing analysis context after the recommendation has been produced.

It is not an input to Strategy and it cannot influence the recommendation action. Its responsibility is transparency and user understanding only.

---

## 5. Scope

### In Scope

- An explanation for every Recommendation produced by Story 1.
- Supporting reasoning associated with that explanation.
- Delivery of explanation and supporting reasoning through the existing Analyze One Asset flow.
- Display of explanation and supporting reasoning in the existing Analyze One Asset workspace.
- Focused coverage at Domain, Application, API, and Web boundaries.

### Out of Scope

- Confidence.
- Risk.
- Watchlists.
- Execution.
- Market scanning.
- Opportunity ranking.
- New analysis endpoints.
- Changes to Story 1 recommendation policy.
- Changes to BUY/HOLD/SELL behavior.

---

## 6. User Journey

1. The user opens Northstar.
2. The user enters one supported asset symbol.
3. The user requests analysis.
4. Northstar returns the existing recommendation.
5. Northstar displays the explanation and supporting reasoning for that recommendation on the same workspace.
6. The user can understand why Northstar made the recommendation and decide whether to trust it.

---

## 7. Repository Responsibilities

### northstar-core

- Define the Domain representation and invariants for a Recommendation Explanation.
- Keep explanation explicitly associated with an existing `Recommendation` and its existing analysis context.
- Implement the Domain behavior that produces the explanation and supporting reasoning.
- Preserve `Strategy` as the sole owner of recommendation policy and action selection.
- Add focused Domain tests proving that explanations support, but do not alter, recommendations.

### northstar-application

- Extend the existing Analyze One Asset orchestration only as needed to obtain the explanation after the Recommendation is produced.
- Return the existing Recommendation together with its explanation as an application outcome.
- Coordinate Domain behavior without interpreting signals, selecting actions, or producing explanation policy outside the Domain.
- Add focused application contract tests using lightweight test doubles where needed.

### northstar-api

- Extend the existing `POST /analyze` response only with the explanation and supporting reasoning available from the Application outcome.
- Keep request behavior, recommendation behavior, and existing error behavior unchanged.
- Translate the Application outcome to the public response without calculating, rewriting, or inventing explanation content.
- Add endpoint coverage for successful explanation delivery and existing failure paths.

### northstar-web

- Replace the current Reason and Supporting Signals placeholders with the explanation and supporting reasoning returned by `POST /analyze`.
- Keep the existing Analyze One Asset workspace and same-page result flow.
- Render only information supplied by the API.
- Preserve the Confidence and Risk placeholders because they remain out of scope for Story 2.
- Add focused user-flow coverage for viewing a recommendation and its explanation.

---

## 8. Acceptance Criteria

Story 2 is accepted when:

- A user can analyze one supported asset through the existing Story 1 workflow.
- The user receives the unchanged Recommendation produced by Story 1.
- The user receives an explanation for that Recommendation.
- The user can see supporting reasoning for the explanation.
- Every Recommendation action currently produced in Story 1 has an explainable outcome.
- Explanation supports the Recommendation and does not determine the action.
- The API, Application, and Web layers do not duplicate Strategy decision logic.
- Confidence, risk, watchlist, execution, and scanner behavior are not introduced.

---

## 9. Definition of Done

Story 2 is complete when Northstar can explain every recommendation produced in Story 1.

Completion requires:

- Domain explanation behavior with focused automated coverage.
- Application orchestration that preserves Domain ownership.
- API delivery of explanation and supporting reasoning through the existing endpoint.
- Web presentation of the returned explanation and supporting reasoning in the existing workspace.
- Story 1 recommendation behavior unchanged.
- Relevant repository lint, format, build, and test checks passing.

---

## 10. Self Review

- Product-first: explanation makes the existing recommendation more useful and trustworthy.
- User-first: the user can understand a recommendation in the same workflow where it is received.
- Story 1 unchanged: recommendation policy and existing analysis behavior remain intact.
- Architecture unchanged: Domain owns business meaning, Application coordinates, API delivers, and Web presents.
- Explanation remains separate from Recommendation: it supports the existing outcome without determining it.

---

STORY 2

Explain Recommendation

Status

Ready for Development

Goal

Every recommendation must be explainable.

---
