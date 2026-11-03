# Embedded themes and output assets

`Sphere10.VisualRenderer` embeds the built-in theme templates, includes, configuration, and assets. Rendering does not create or populate a theme directory.

```csharp
var renderer = new HtmlRenderer(new ThemeCatalog(new ThemeOptions {
    ThemesDirectory = optionalOverrideDirectory
}));
var result = renderer.Render(document, new RenderOptions {
    Themes = ["default"],
    Environment = RenderEnvironment.Offline,
    AssetBaseUrl = "../render-assets"
});
```

Omit `ThemeCatalog` or `ThemesDirectory` to use the embedded defaults. A configured directory need not exist. LocalNotion additionally deploys missing built-in files to its configured themes path, normally `.localnotion/themes`, on repository creation and opening. Existing files are preserved. This deployment belongs to Core; the standalone renderer does not perform it.

To override one file, create the corresponding named-theme path, for example `.localnotion/themes/default/paragraph.html`. The remaining files, including `.config.json`, fall back to the assembly. Empty files count as overrides. In the standalone renderer, removing an override reveals the embedded version on the next independent render or render batch. LocalNotion also restores that missing built-in file on its next repository open. A session keeps the bytes it read, so editing files during later renders cannot change an earlier result. `HtmlRenderer` calls `ThemeCatalog.CreateSnapshot()` per render; a catalog that is already a snapshot returns itself and reuses sessions for the same ordered themes, environment, mode and asset base URL. Pass `catalog.CreateSnapshot()` to the renderer when rendering a group of documents against one consistent set of overrides. Direct `CreateSession` calls on a catalog with a mutable override directory refresh independently.

A custom theme needs its own `.config.json` and can inherit a built-in theme:

```json
{
  "type": "html",
  "base": "default",
  "tokens": {
    "BrandName": { "offline": "Example", "online": "Example" }
  }
}
```

For each named theme, disk files override the matching embedded files. The first selected theme contributes its complete base chain, applied from the oldest base through the selected theme. Each later selected theme overlays only its own files and tokens, in selection order; its base chain is not applied again. An explicitly selected base later in the list therefore overrides earlier values with its own definitions. This preserves the original renderer's cascade: decorative themes can replace their specific includes without resetting earlier carousel, column, or typography templates. File origin never outranks these theme priorities.

The existing template selection order remains: mode folder plus environment variant, mode folder generic, root generic, root environment variant. In particular, `readonly/paragraph.html` is selected before `paragraph.html`.

Missing required templates, invalid configuration, unknown themes, and inheritance/include cycles report `ThemeException`; invalid override files are not silently ignored. File names and token names use ordinal comparison. Preserve their spelling across operating systems.

Previously extracted theme copies remain ordinary overrides. They can mask newer embedded defaults and corrected built-in template references. The library never deletes or rewrites them automatically.

Full-document root Page and database wrappers begin with `<!DOCTYPE html>` so HTML parsers select standards mode. Fragments (`RenderFrame = false` or `RenderSnippet`) do not receive that document preamble. Existing extracted wrappers may predate this correction and continue to select quirks mode. A selected modifier can override `readonly/page.online.html` or `readonly/page.offline.html` through the normal template cascade while retaining the matching `{include://page.online.html}` or `{include://page.offline.html}` for the existing inner head/body composition; changing a shared `.root` override affects all documents using it.

## Page covers and header heights

The default cover keeps its existing `35vh` height, `200px` minimum, centered background image and full-height Bootstrap title wrapper. Its native document subtitle retains the normal outside `page_subtitle.html` location. Cover presentation can be changed through the same ordered Theme composition used by other native headers; it does not require rewriting the shared cover stylesheet.

Select `header_cover` for a white, naturally sized title/optional Subtitle group centered with Bootstrap flex utilities. Its cover template owns a definite viewport height through `cover_height.inc`, defaulting to `35vh` to match the existing stock cover stylesheet. Select an existing size modifier after it: `header_tiny` supplies `15vh`, `header_small` supplies `25vh`, `header_large` supplies `75vh`, and `header_full` supplies `100vh`. The normal cover minimum height still applies. For example, `Themes = ["cms", "header_cover", "header_large"]` composes the CMS page layout with a large centered cover. The size modifiers retain their original `header_height.inc` classes for existing carousel Headers; the new cover include does not change that composition.

`header_cover` uses the supported configuration token `page_subtitle_placement` with offline/online values of `cover`. When an actual native cover and `TitleOnCover` are present, this opts the native document Subtitle into the cover group. The renderer supplies `{text}` and optional `{cover_subtitle}` to `page_cover_title.html`; the fragment `page_cover_subtitle.html` receives the rendered rich text as both `{subtitle}` and `{description}`. A missing or empty native Subtitle emits no fragment. The title stays centered by itself, while a title/Subtitle pair centers as one natural-height group and moves the title upward. Preserve `page-cover-title`, its `data-toc-skip` attribute and the Theme's cover-only lettering. Body headings and Typed content stay in the body; they are not inferred banner subtitles.

The renderer does not inspect rendered HTML or identify a Theme by name to decide this placement. Default and legacy Themes without the explicit token keep their outside subtitle. A Theme opting into cover placement owns the matching title/Subtitle fragments. Without an actual cover or when `TitleOnCover` is false, native subtitles remain outside even if that Theme is selected. `TitleOnCover` continues to suppress the normal body title without a native cover, because a parent CMS or custom frame may already own it.

Previously extracted files remain ordinary overrides and can mask newer built-in Theme files. Adopt the selected `header_cover` files and matching size include through the normal Theme installer when refreshing a deployment. Its markup and Bootstrap utilities work with the existing shared stylesheet; the default cover stylesheet and carousel height behavior remain unchanged. A local renderer update does not itself update an independently hosted asset URL.

## Presentation Themes and document behavior

Themes select ordered template, token and resource overrides. They do not interpret source metadata or CMS Tags. A source adapter translates behavior controls into explicit visual-model properties before rendering. For example, Local Notion's `@HideEmbeddedPages` Tag sets `DocumentBlock.HideEmbeddedPages`; it does not add a Theme of the same name.

```csharp
var document = new DocumentBlock {
	Themes = ["default"],
	HideEmbeddedPages = true,
	Children = [
		new ReferenceBlock {
			IsEmbeddedPage = true,
			Reference = new Reference { Url = "/manual", Label = "Contained manual" }
		},
		new ParagraphBlock { Text = [new TextInline { Text = "Read a chapter", Url = "/manual#chapter" }] }
	]
};
```

The automatic contained-manual widget is omitted from HTML while the authored chapter link remains visible. Render the destination independently with its normal content and presentation Themes. The flag does not remove model/source nodes, change publication or access policy, suppress other child content or hide the destination's document body. It defaults to `false` and leaves plain-text output unchanged.

The nearest containing `DocumentBlock` determines suppression, independently of Theme selection. A nested document owns its own flag even if it inherits its parent's presentation Themes. The behavior works in online/offline and editable/read-only HTML without JavaScript removal, route-specific conditions or `display:none` rules.

Visible embedded references still pass normal reference HTML through `embedded_page.html`. The default slot is `{contents}`; a custom Theme may style it through the normal cascade. This presentation hook is separate from the document's visibility control. An earlier development build supplied a `hide_embedded_pages` suppression Theme; consumers of that build must explicitly move the behavioral selection to the canonical `@HideEmbeddedPages` source Tag or `HideEmbeddedPages` model flag and retire only that obsolete Theme override. The renderer does not silently reinterpret a Theme name as a Tag.

## Checklist rich text

The `bullet_check` modifier keeps its check badge separate from a single shrinkable content wrapper. Rich-text runs and hyperlinks stay in normal inline flow, and nested `.ln-bulleted-list-item-children` remain inside that wrapper below the item text. The wrapper uses `min-width:0` and `overflow-wrap:anywhere` so narrow layouts can wrap long linked labels and technical tokens without clipping them or forcing an unbroken flex row.

Existing extracted `bullet_check/bulleted_list_item.html` files remain ordinary project overrides and can mask this correction. Update that one override through the normal theme workflow when adopting the corrected layout; installing a newer renderer preserves an existing override. A template-only project override does not update the installed renderer binary.

## Publishing the result

An `HtmlRenderResult` contains `Html` and `Assets`. Each asset has:

- `RelativePath`: a path such as `assets/{sha256}/resources/local-notion/css/ln.css`.
- `ContentType`: the media type to return when serving it.
- `Content`: the in-memory bytes of that render's resource snapshot.

The HTML's asset URLs are `AssetBaseUrl` plus those relative paths; when no base is supplied, they are relative to the rendered document. A web host can serve `Assets` from memory using their paths and content types. A static exporter can write them under its chosen output asset directory. LocalNotion's generated asset output is separate from its theme override directory.

The bundle hash covers the effective resource paths and bytes. Different overrides cannot overwrite resources used by an earlier document. Place custom assets and their relative dependencies under the theme's `resources/` tree. Each bundle preserves complete effective resource subtrees, including MathJax fonts and modules and Prism's bundled language grammars. Reuse identical bundle paths when publishing several documents.

`RenderEnvironment.Online` uses each asset's owning theme `online_url`, whether that file comes from disk or the assembly. Existing extracted theme folders and local file overrides do not disable CDN links. Inherited files retain their owning theme's URL: CMS scripts use the CMS CDN directory, while the inherited LocalNotion script uses the default theme directory. Template and include contents still prefer disk overrides. To change a remotely served asset, publish it at the configured URL or configure the theme to use your own hosted URL. Offline rendering, and assets whose owning theme has no online URL, use the emitted resource bundle and `AssetBaseUrl`.

The no-disk library contract is HTML plus in-memory resources. An offline static page requires the emitted resource files beside its output, or a host that serves the same bytes. It is not a promise of self-contained single-file HTML. External media and social embeds retain their own network requirements.

Mermaid code blocks load a separate, pinned internal runtime bundle with no CDN owner. Their scripts, stylesheet and license files are returned in `Assets` in **both** environments and referenced once per render. Serve these assets even when the other online theme dependencies use CDNs. Ordinary code and documents without Mermaid do not load that runtime. See [Mermaid diagrams](mermaid.md).

Run theme regressions with:

```powershell
dotnet test tests/Sphere10.VisualRenderer.Tests/Sphere10.VisualRenderer.Tests.csproj
```
