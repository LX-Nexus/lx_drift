# 5. UI Development Guidelines

When contributing to Lexi's UI, please adhere to the following guidelines:

### 5.1. Understanding Flet Controls and Layout

*   **Familiarize yourself with Flet documentation**: Before making significant UI changes, review the official [Flet documentation](https://flet.dev/docs/) to understand available controls, their properties, and layout options.
*   **Use appropriate controls**: Select Flet controls that best fit the UI element's purpose and interaction. For example, `ft.Container` for grouping and styling, `ft.Text` for static text, `ft.TextField` for input.
*   **Responsive Design**: Design UI components to be responsive. Flet's layout system (e.g., `Row`, `Column`, `Container` with `expand` property, `ResponsiveRow`) should be used to ensure the UI adapts gracefully to different screen sizes and orientations.

### 5.2. Working with Images and Animations

*   **Existing Base64 Assets**: For the Lexi icon and avatar, continue to use the existing Base64 approach. If these assets need to be updated, convert the new image/GIF to Base64 and replace the content of `icon_base64.txt` or `avatar_base64.txt` respectively. You can use online tools or command-line utilities (e.g., `base64 -i image.png > image_base64.txt`) for conversion.
*   **New Static Images**: For any *new* static images that are small and integral to the application's self-contained nature (e.g., small icons, logos that are always displayed), consider converting them to Base64 and embedding them similarly. However, be mindful of the file size overhead.
*   **Larger Images/Dynamic Content**: For larger images, user-uploaded content, or images that might change frequently, consider alternative approaches that Flet supports, such as loading images from local paths or URLs. Flet's `ft.Image` control can directly take a file path or a URL as its `src` property. This avoids the Base64 overhead for assets that don't benefit from it.
*   **Animations**: For complex or longer animations, evaluate if GIF is the most suitable format. While Base64 embedded GIFs work for simple animations, consider optimized video formats or Lottie animations if Flet supports them in the future, especially if performance becomes an issue.

### 5.3. Styling and Theming

*   **Consistent Styling**: Maintain a consistent visual style throughout the application. Use Flet's theming capabilities (if available and implemented) or consistently apply `ft.TextStyle` and `ft.Container` properties for colors, fonts, padding, and margins.
*   **Color Palette**: Adhere to Lexi's defined color palette. If no formal palette exists, establish one and document it.

### 5.4. Code Quality and Maintainability

*   **Modularity**: Break down complex UI into smaller, reusable Flet controls or functions. This improves readability and maintainability.
*   **Clear Naming Conventions**: Use descriptive names for variables, functions, and controls.
*   **Comments**: Add comments where necessary to explain complex logic or non-obvious UI decisions.
*   **Error Handling**: Implement robust error handling for UI operations, especially when dealing with external resources or user input.
*   **Performance**: Be mindful of UI performance. Avoid unnecessary `page.update()` calls and optimize complex layouts.
