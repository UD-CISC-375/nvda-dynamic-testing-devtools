# CISC 375 Branching Convention Guide

Fall 2026 DevOps Team

## Background

In order to coordinate effective collaboration between teams for NVDA development, DevOps team has established a convention for how Git branches should be structured and named. This convention is enforced by GitHub rulesets to ensure that development proceeds smoothly and efficiently. This guide provides an overview of this convention, details about convention enforcement, and a list of best practices to ensure that you can combine your work with your peers' without issue.

## Structure

There are exactly FOUR categories of branches under this convention:

1. **Master:** The default and primary branch of the project. This branch should ALWAYS contain fully-functional, release-ready, NVDA-guideline-adhering code.
2. **Class-Scoped Goals:** These branch directly off of Master. They should encapsulate a development goal that is shared by multiple teams in the class. These branches should ALWAYS contain fully-functional, release-ready code.
3. **Team-Scoped Goals:** These branch off of Class-Scoped Goals only. They should encapsulate a development goal that is entirely contained within a single team. These branches should ALWAYS contain fully-functional code.
4. **Individually-Scoped Goals:** These branch off of Team-Scoped Goals only. They should encapsulate a development goal that is being worked towards by a single developer, or a subset of developers within the same team. These branches have no requirements for the code they contain.

This structure is NOT enforced by GitHub rules inherently, as such protections require an Enterprise subscription. Please make sure to adhere to our naming convention to allow the GitHub rulesets to protect this structure.

## Naming

Each of the four branch categories specified above has a corresponding naming convention that they should follow:

1. **Master:** The default branch will always be called master.\*
2. **`<class-goal-name>/master`:** Names for Class-Scoped Goals should include an all-lowercase, hyphenated description of the goal shared by multiple teams, followed by EXACTLY `/master`. This ensures that this goal name can be used as a prefix in other branch names.
3. **`<class-goal-name>/<L>_<team-goal-name>/master`:** Names for Team-Scoped Goals should include the name of the Class-Scoped Goal it is branching off of, followed by the lowercase letter of the team whose goal the branch belongs to and a trailing underscore, followed by an all-lowercase, hyphenated description of the goal being worked towards by team `<L>`, followed by EXACTLY `/master`. This ensures that this goal name can be used as a prefix in other branch names.
4. **`<class-goal-name>/<L>_<team-goal-name>/<anything>`:** Names for Individually-Scoped Goals should include the name of the Class-Scoped Goal it is branching off of, followed by the lowercase letter of the team whose goal the branch belongs to and a trailing underscore, followed by an all-lowercase, hyphenated description of the goal being worked towards by team `<L>`, followed by any name the involved developers would like. The `<anything>` section can include any text that the developers would like.

For example, say the class wants to add DOM parsing as a feature. First, a branch called `parse-dom-tree/master` would be split off of the Master branch. Team Y would then make a branch called `parse-dom-tree/y_integrate-dom-api/master`, and a member of Team Y could then make a branch called `parse-dom-tree/y_integrate-dom-api/testingImplementation2`.

---

\* **Errata:** We are still using `master` in this convention for now, because the GitHub rulesets protect branches named `*/master`. We will swap to `main` once the branching rulesets are updated. (The framework, webpage, and devtools repos already name their default branch `main`.)
