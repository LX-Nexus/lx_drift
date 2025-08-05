# 3. UI Architecture with Flet

Lexi's user interface is built with [Flet](https://flet.dev/), a framework that enables building reactive UIs in Python. Flet applications are composed of controls (widgets) that form a visual tree. Changes to this tree are efficiently rendered to the underlying UI platform (web, desktop, or mobile).

Key aspects of Flet's usage in Lexi:

*   **Reactive Programming**: The UI responds to state changes. When data changes, the relevant UI components are automatically updated.
*   **Controls**: Flet provides a rich set of pre-built UI controls (e.g., `ft.Text`, `ft.Image`, `ft.Container`, `ft.ProgressBar`) that are used to construct the application's layout and interactive elements.
*   **Page Updates**: UI changes are typically triggered by calling `page.update()`, which sends the updated control tree to the Flet client for rendering.
