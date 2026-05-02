# Alternative Guide - Power Apps Canvas App Rebuild

This is an alternative path only. The active deployment uses Azure Static Web Apps and the hosted HTML calculator instead of rebuilding the app from scratch in Power Apps.

# JE Scoring Calculator - Power Apps Canvas App Build Sheet

This rebuilds the current HTML calculator as a Power Apps Canvas app.

## App Setup

Create a new **Canvas app** in Power Apps.

Recommended layout:

- Tablet layout if this will be used by HR/admins on desktop.
- Phone layout if this will mainly be used from Teams/mobile.

Set the main screen name to:

```text
scrCalculator
```

## Controls

Add these controls to `scrCalculator`.

| Control | Name | Type |
|---|---|---|
| Role title input | `txtRoleTitle` | Text input |
| Category selector | `ddCategory` | Dropdown |
| Team accountability selector | `ddTeam` | Dropdown |
| Scope score | `radScope` | Radio |
| Complexity score | `radComplexity` | Radio |
| Autonomy score | `radAutonomy` | Radio |
| Stakeholder score | `radStakeholder` | Radio |
| Expertise score | `radExpertise` | Radio |
| Total score display | `lblTotalScore` | Label |
| Grade band display | `lblGradeBand` | Label |
| Profile display | `lblProfile` | Label |
| Category display | `lblCategoryResult` | Label |
| Status/note display | `lblNotes` | Label |
| Reset button | `btnReset` | Button |

## Dropdown Formulas

Set `ddCategory.Items` to:

```powerfx
[
    "Technical",
    "Operational",
    "Sales",
    "Hybrid: Technical + Sales",
    "Hybrid: Technical + Operational",
    "Hybrid: Sales + Operational"
]
```

Set `ddTeam.Items` to:

```powerfx
[
    "Individual contributor (IC)",
    "Guides others (no direct reports)",
    "Manages a team (direct reports)"
]
```

Set each scoring radio control's `Items` property to:

```powerfx
[1, 2, 3, 4, 5]
```

Use this for:

- `radScope.Items`
- `radComplexity.Items`
- `radAutonomy.Items`
- `radStakeholder.Items`
- `radExpertise.Items`

## Reusable Score Formula

Power Apps does not have local named helper formulas in every tenant, so paste the relevant `With(...)` formula into each display control below.

The score table used by all formulas is:

```powerfx
Table(
    {Score: radScope.Selected.Value},
    {Score: radComplexity.Selected.Value},
    {Score: radAutonomy.Selected.Value},
    {Score: radStakeholder.Selected.Value},
    {Score: radExpertise.Selected.Value}
)
```

## Result Formulas

Set `lblTotalScore.Text` to:

```powerfx
With(
    {
        Scores: Table(
            {Score: radScope.Selected.Value},
            {Score: radComplexity.Selected.Value},
            {Score: radAutonomy.Selected.Value},
            {Score: radStakeholder.Selected.Value},
            {Score: radExpertise.Selected.Value}
        )
    },
    With(
        {
            ScoredCount: CountIf(Scores, !IsBlank(Score)),
            RawTotal: Sum(Scores, Coalesce(Score, 0))
        },
        If(
            ScoredCount < 3,
            "Score at least 3 criteria (" & ScoredCount & "/5 scored)",
            Text(If(ScoredCount < 5, Round(RawTotal / ScoredCount * 5, 0), RawTotal))
        )
    )
)
```

Set `lblGradeBand.Text` to:

```powerfx
With(
    {
        Scores: Table(
            {Score: radScope.Selected.Value},
            {Score: radComplexity.Selected.Value},
            {Score: radAutonomy.Selected.Value},
            {Score: radStakeholder.Selected.Value},
            {Score: radExpertise.Selected.Value}
        )
    },
    With(
        {
            ScoredCount: CountIf(Scores, !IsBlank(Score)),
            RawTotal: Sum(Scores, Coalesce(Score, 0))
        },
        With(
            {
                ProjectedScore: If(ScoredCount < 5, Round(RawTotal / ScoredCount * 5, 0), RawTotal)
            },
            If(
                ScoredCount < 3,
                "",
                Switch(
                    true,
                    ProjectedScore <= 9, "B1-B2",
                    ProjectedScore <= 13, "B3-C1",
                    ProjectedScore <= 16, "C2-C3",
                    ProjectedScore <= 19, "C4-C5",
                    ProjectedScore <= 21, "D1",
                    "D2-D3"
                )
            )
        )
    )
)
```

Set `lblProfile.Text` to:

```powerfx
With(
    {
        Scores: Table(
            {Score: radScope.Selected.Value},
            {Score: radComplexity.Selected.Value},
            {Score: radAutonomy.Selected.Value},
            {Score: radStakeholder.Selected.Value},
            {Score: radExpertise.Selected.Value}
        )
    },
    With(
        {
            ScoredCount: CountIf(Scores, !IsBlank(Score)),
            RawTotal: Sum(Scores, Coalesce(Score, 0))
        },
        With(
            {
                ProjectedScore: If(ScoredCount < 5, Round(RawTotal / ScoredCount * 5, 0), RawTotal)
            },
            If(
                ScoredCount < 3,
                "",
                Switch(
                    true,
                    ProjectedScore <= 9, "Analyst / Junior Specialist",
                    ProjectedScore <= 13, "Senior Analyst / Specialist",
                    ProjectedScore <= 16, "Senior Specialist / Lead",
                    ProjectedScore <= 19, "Principal Specialist / Senior Lead",
                    ProjectedScore <= 21, "Manager / Senior Principal",
                    "Senior Manager / Director"
                )
            )
        )
    )
)
```

Set `lblCategoryResult.Text` to:

```powerfx
If(IsBlank(ddCategory.Selected.Value), "-", ddCategory.Selected.Value)
```

Set `lblNotes.Text` to:

```powerfx
With(
    {
        Scores: Table(
            {Score: radScope.Selected.Value},
            {Score: radComplexity.Selected.Value},
            {Score: radAutonomy.Selected.Value},
            {Score: radStakeholder.Selected.Value},
            {Score: radExpertise.Selected.Value}
        )
    },
    With(
        {
            ScoredCount: CountIf(Scores, !IsBlank(Score)),
            RawTotal: Sum(Scores, Coalesce(Score, 0))
        },
        With(
            {
                ProjectedScore: If(ScoredCount < 5, Round(RawTotal / ScoredCount * 5, 0), RawTotal),
                TeamNote: If(
                    ddTeam.Selected.Value = "Manages a team (direct reports)" && ProjectedScore < 20,
                    "Team management detected - consider bumping one grade level toward D1.",
                    If(
                        ddTeam.Selected.Value = "Individual contributor (IC)" && ProjectedScore >= 20,
                        "No direct reports - D-level may be too high; confirm scope justifies this.",
                        ""
                    )
                ),
                HybridNote: If(
                    StartsWith(ddCategory.Selected.Value, "Hybrid"),
                    "Hybrid role: anchor grade on the primary category, note secondary as context.",
                    ""
                )
            },
            If(
                ScoredCount < 3,
                "",
                Trim(TeamNote & Char(10) & HybridNote)
            )
        )
    )
)
```

## Reset Button

Set `btnReset.Text` to:

```powerfx
"Reset"
```

Set `btnReset.OnSelect` to:

```powerfx
Reset(txtRoleTitle);
Reset(ddCategory);
Reset(ddTeam);
Reset(radScope);
Reset(radComplexity);
Reset(radAutonomy);
Reset(radStakeholder);
Reset(radExpertise)
```

## Optional: Save Results

If you want the app to save each scoring result, create a SharePoint list or Dataverse table with these columns:

| Column | Type |
|---|---|
| RoleTitle | Text |
| Category | Text |
| TeamAccountability | Text |
| ScopeScore | Number |
| ComplexityScore | Number |
| AutonomyScore | Number |
| StakeholderScore | Number |
| ExpertiseScore | Number |
| TotalJEScore | Number |
| GradeBand | Text |
| Profile | Text |
| Notes | Multiple lines of text |

Then add a `Save` button and use `Patch(...)` to write the record.

## Original Scoring Rules

| Score Range | Grade | Profile |
|---|---|---|
| 5-9 | B1-B2 | Analyst / Junior Specialist |
| 10-13 | B3-C1 | Senior Analyst / Specialist |
| 14-16 | C2-C3 | Senior Specialist / Lead |
| 17-19 | C4-C5 | Principal Specialist / Senior Lead |
| 20-21 | D1 | Manager / Senior Principal |
| 22-25 | D2-D3 | Senior Manager / Director |
