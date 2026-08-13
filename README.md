
# 👋 Hola, soy Facu 👋
### Desarrollor de Software | Java • Spring • React • PostgreSQL • Docker
 ### 🧰 Tecnologías y herramientas
<p>
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/java/java-original.svg" width="45"/>
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/spring/spring-original.svg" width="45"/>
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/react/react-original.svg" width="45"/>
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/javascript/javascript-original.svg" width="45"/>
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/postgresql/postgresql-original.svg" width="45"/>
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/docker/docker-original.svg" width="50"/>
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/figma/figma-original.svg" width="45"/>
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/python/python-original.svg" width="45"/>
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/vscode/vscode-original.svg" width="45" height="45" alt="VS Code" />
  <img src="https://registry.npmmirror.com/@lobehub/icons-static-png/latest/files/dark/cursor.png" width="45" height="45" alt="Cursor AI" />
   <img src="https://registry.npmmirror.com/@lobehub/icons-static-png/latest/files/dark/claude-color.png" width="45" height="45" alt="Claude AI" />
   // @ts-check

import { renderTopLanguages } from "../src/cards/top-languages.js";
import { guardAccess } from "../src/common/access.js";
import {
  CACHE_TTL,
  resolveCacheSeconds,
  setCacheHeaders,
  setErrorCacheHeaders,
} from "../src/common/cache.js";
import {
  MissingParamError,
  retrieveSecondaryMessage,
} from "../src/common/error.js";
import { parseArray, parseBoolean } from "../src/common/ops.js";
import { renderError } from "../src/common/render.js";
import { fetchTopLanguages } from "../src/fetchers/top-languages.js";
import { isLocaleAvailable } from "../src/translations.js";

// @ts-ignore
export default async (req, res) => {
  const {
    username,
    hide,
    hide_title,
    hide_border,
    card_width,
    title_color,
    text_color,
    bg_color,
    theme,
    cache_seconds,
    layout,
    langs_count,
    exclude_repo,
    size_weight,
    count_weight,
    custom_title,
    locale,
    border_radius,
    border_color,
    disable_animations,
    hide_progress,
    stats_format,
  } = req.query;
  res.setHeader("Content-Type", "image/svg+xml");

  const access = guardAccess({
    res,
    id: username,
    type: "username",
    colors: {
      title_color,
      text_color,
      bg_color,
      border_color,
      theme,
    },
  });
  if (!access.isPassed) {
    return access.result;
  }

  if (locale && !isLocaleAvailable(locale)) {
    return res.send(
      renderError({
        message: "Something went wrong",
        secondaryMessage: "Locale not found",
        renderOptions: {
          title_color,
          text_color,
          bg_color,
          border_color,
          theme,
        },
      }),
    );
  }

  if (
    layout !== undefined &&
    (typeof layout !== "string" ||
      !["compact", "normal", "donut", "donut-vertical", "pie"].includes(layout))
  ) {
    return res.send(
      renderError({
        message: "Something went wrong",
        secondaryMessage: "Incorrect layout input",
        renderOptions: {
          title_color,
          text_color,
          bg_color,
          border_color,
          theme,
        },
      }),
    );
  }

  if (
    stats_format !== undefined &&
    (typeof stats_format !== "string" ||
      !["bytes", "percentages"].includes(stats_format))
  ) {
    return res.send(
      renderError({
        message: "Something went wrong",
        secondaryMessage: "Incorrect stats_format input",
        renderOptions: {
          title_color,
          text_color,
          bg_color,
          border_color,
          theme,
        },
      }),
    );
  }

  try {
    const topLangs = await fetchTopLanguages(
      username,
      parseArray(exclude_repo),
      size_weight,
      count_weight,
    );
    const cacheSeconds = resolveCacheSeconds({
      requested: parseInt(cache_seconds, 10),
      def: CACHE_TTL.TOP_LANGS_CARD.DEFAULT,
      min: CACHE_TTL.TOP_LANGS_CARD.MIN,
      max: CACHE_TTL.TOP_LANGS_CARD.MAX,
    });

    setCacheHeaders(res, cacheSeconds);

    return res.send(
      renderTopLanguages(topLangs, {
        custom_title,
        hide_title: parseBoolean(hide_title),
        hide_border: parseBoolean(hide_border),
        card_width: parseInt(card_width, 10),
        hide: parseArray(hide),
        title_color,
        text_color,
        bg_color,
        theme,
        layout,
        langs_count,
        border_radius,
        border_color,
        locale: locale ? locale.toLowerCase() : null,
        disable_animations: parseBoolean(disable_animations),
        hide_progress: parseBoolean(hide_progress),
        stats_format,
      }),
    );
  } catch (err) {
    setErrorCacheHeaders(res);
    if (err instanceof Error) {
      return res.send(
        renderError({
          message: err.message,
          secondaryMessage: retrieveSecondaryMessage(err),
          renderOptions: {
            title_color,
            text_color,
            bg_color,
            border_color,
            theme,
            show_repo_link: !(err instanceof MissingParamError),
          },
        }),
      );
    }
    return res.send(
      renderError({
        message: "An unknown error occurred",
        renderOptions: {
          title_color,
          text_color,
          bg_color,
          border_color,
          theme,
        },
      }),
    );
  }
};

   
  




</p>

