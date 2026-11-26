# 16. Angular templates and HTML structure

An Angular template is the view markup for a component. It is based on HTML, so good HTML structure, semantic elements, and accessible markup still matter. Angular adds its own compilation and template syntax on top of that HTML.

This chapter establishes the structure and rules of a template. Later lessons focus on interpolation, bindings, events, and control flow.

## Learning goals

After this lesson, you should be able to:

- Explain how a component template fits inside an Angular application.
- Write readable, well-nested HTML in an Angular template.
- Use semantic elements and meaningful document structure.
- Recognize Angular template features that differ from ordinary HTML.
- Understand comments, whitespace, self-closing component tags, and unsupported script elements.
- Diagnose basic template parsing errors without confusing template source with the rendered DOM.

## 1. A component template is an HTML-based view

Every component has a template. It describes the elements Angular should render for that component. A template can be written inline in component metadata or in a separate HTML file.

A template is not the same as the full document shell. The workspace's index.html contains the document and the root component host; Angular renders the root component's template inside that host.

For example, a workspace index.html might contain:

~~~html
<body>
  <app-root></app-root>
</body>
~~~

The app-root element matches the root component selector. The root component template supplies the application view inside it.

## 2. Use valid, readable HTML structure

Angular templates use HTML nesting. Place child elements inside their intended parent, close non-void elements correctly, and keep indentation consistent.

~~~html
<main>
  <section aria-labelledby="upcoming-title">
    <h1 id="upcoming-title">Upcoming events</h1>
    <p>Choose an event to see its details.</p>

    <ul>
      <li>Angular study session</li>
      <li>Template practice</li>
    </ul>
  </section>
</main>
~~~

Good indentation makes nesting visible to readers. It also makes missing or misplaced closing tags easier to find.

HTML void elements, such as img, input, and br, do not have a closing tag. Ordinary elements such as main, section, p, and li must be properly closed. Angular's template parser reports many malformed structures during compilation.

## 3. Prefer semantic elements

Choose HTML elements for their meaning and built-in behavior:

- Use main for the page's primary content.
- Use nav for navigation.
- Use section for a themed grouping, usually with a heading.
- Use article for a self-contained piece of content.
- Use ul or ol for lists.
- Use button for an action and a for navigation.
- Use headings in a meaningful hierarchy.

Semantic HTML helps browsers, assistive technology, and other developers understand the page. Add labels and text alternatives where controls and images need them. A div can group content, but it has no native meaning or keyboard behavior by itself.

A custom Angular component should also be placed in an appropriate surrounding structure. Its selector does not automatically provide the semantics of a main, button, or heading element.

## 4. Angular compiles templates

Angular reads template source and compiles it as part of the application build. The resulting view includes Angular's rendering behavior and may contain framework-generated DOM nodes used for dynamic content. The Elements panel in browser developer tools shows the rendered DOM, which may not match the source file character for character.

Angular template syntax includes features such as:

- Text interpolation using double curly braces.
- Property and attribute bindings using bracket syntax.
- Event listeners using parentheses.
- Built-in control flow blocks beginning with the at-sign.
- Component, directive, and pipe references.

These markers have their own meaning in an Angular template. The next chapters explain each feature. For now, remember that a template is HTML with Angular syntax and compile-time checks, not an arbitrary JavaScript string inserted into the page.

## 5. Angular-specific differences from ordinary HTML

Most HTML markup is valid in a template, but there are a few important differences:

- Angular component and directive elements can use self-closing syntax, such as app-banner, in supported Angular versions. Self-closing component and directive tags are supported starting in Angular 16.
- Angular templates do not support script elements. Put application code in TypeScript files and configure required scripts through the workspace build setup.
- HTML comments in the template source are not emitted as visible page content.
- Angular may collapse or remove whitespace it considers unnecessary.
- The at-sign has special meaning for newer template syntax. To display a literal at-sign in template text, write the HTML entity &commat; or &#64;.

Continue using normal HTML element rules. A self-closing Angular component is not a reason to omit closing tags from ordinary non-void HTML elements.

## 6. Whitespace and comments

Browsers collapse runs of whitespace in ordinary text in many contexts. Angular also removes or collapses whitespace it considers unnecessary while compiling a template. Indentation and line breaks used only to format markup generally do not become extra visible spacing.

Write visible text naturally:

~~~html
<p>
  A clear sentence can wrap across lines in the source.
</p>
~~~

If a specific space is meaningful between adjacent inline elements, use an explicit HTML entity where necessary. Angular also provides the &ngsp; entity for a space that should be preserved in compiled output.

Avoid enabling preserveWhitespaces for an entire component unless the view truly requires it, such as a whitespace-sensitive display. It can retain extra text nodes and affect rendering work.

HTML comments can help explain template source to maintainers, but Angular does not render them as page text. Do not put content that users must see only in an HTML comment.

## 7. Template elements and rendered elements

Some tags in Angular templates represent framework behavior rather than ordinary visible HTML elements. Components add their host elements; structural template features can introduce anchors or comment nodes; and special template constructs can define content that is not rendered immediately.

Use browser developer tools to inspect the resulting DOM when layout or rendering is unexpected. Compare the rendered elements with the component template and the Angular diagnostics rather than assuming the source file is copied directly into the browser.

## 8. Keep template responsibilities clear

Templates describe view structure and use Angular syntax to connect that structure with component state. Put application logic in TypeScript, reusable non-view behavior in appropriate services, and markup in the component template.

Keep the template readable:

- Use indentation to show parent-child structure.
- Break large sections into meaningful components when that improves clarity.
- Avoid deeply nested generic containers when semantic elements express the structure.
- Keep static text and labels understandable.
- Use the next lessons to add dynamic values, bindings, and interaction instead of mixing those concerns into raw HTML.

## Common mistakes

- Treating a component template as a complete HTML document with its own html, head, and body shell.
- Using a closing tag for a void element such as img or input.
- Leaving an ordinary element unclosed or nesting it in a way the HTML parser cannot interpret.
- Using div elements where a native semantic element communicates the purpose better.
- Assuming the browser's rendered DOM exactly matches the template source.
- Putting script elements or executable application code inside a component template.
- Expecting HTML comments to appear as visible text.
- Relying on indentation whitespace to create exact spacing in the rendered page.
- Writing a literal at-sign in a location where Angular interprets it as template syntax without escaping it.
- Using Angular syntax without including the component, directive, or pipe dependency required by that template.

## Practical checklist

- [ ] The component template contains only the view, not a duplicate document shell.
- [ ] Elements are properly nested and void elements have no closing tags.
- [ ] Semantic elements express the page structure and actions.
- [ ] Headings and labels make the view understandable.
- [ ] Angular-specific syntax is intentional and will be covered in the relevant lesson.
- [ ] No script element is embedded in the template.
- [ ] Whitespace and comments are not being mistaken for visible content.
- [ ] The rendered DOM has been inspected when the visual structure is unexpected.

## Next lesson

Continue with [Interpolation and template expressions](017-interpolation-and-template-expressions.md).

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="015-selectors-and-the-component-host-element.md">015 - 15. Selectors and the component host element</a></td><td><a href="../qnas/016-angular-templates-and-html-structure.md">Q&amp;A</a></td><td><a href="../examples/016-angular-templates-and-html-structure.md">Example</a></td><td><a href="017-interpolation-and-template-expressions.md">017 - 17. Interpolation and template expressions</a></td>
</tr>
  </tbody>
</table>
