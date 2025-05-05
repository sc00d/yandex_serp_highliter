<h1>Подсветка рекламы и своих сайтов в выдаче Яндекса на выдаче для ПК и для мобильных устройств</h1>


![image](https://github.com/user-attachments/assets/70c71f12-4a74-4fce-8cae-046eff35dd4d)

![image](https://github.com/user-attachments/assets/3873bf79-7b1a-45d3-a64f-387e05ad3719)

![image](https://github.com/user-attachments/assets/a30a4cdf-0f3c-436b-856f-ae702d5ffad4)



Решение для тех, кому надоело всматриваться в невидимые подписи "Реклама", (а теперь и значки ₽), которые притаились за хлебным крошками или где-то в углу. 
Со скриптом удобно смотреть где этот ваш "ТОП-10" органики находится в реальности у юзера и понимать почему CTR такой низкий =(

— Работает в Моб и ПК выдаче
— Маркирует рекламу, которая появляется внутри органики при клике по одному из результатов

Чтобы пользоваться в инкогнито - просто разрешите расширению работать в нем (ПКМ по расширению - Управление расширениями - Разрешить использование в режиме инкогнито)

<h2>Об авторе</h2>

Автор: https://t.me/sc00d
Канал: https://t.me/seregaseo

<h2>Как пользоваться?</h2>
Удобней всего использовать расширение https://chromewebstore.google.com/detail/user-javascript-and-css/nbhcbdghjpllgmfilhnhkllmkecfmpld

Добавляем новое правило для URL Pattern: 
<code>

https://www.yandex.ru/*, https://www.ya.ru/search*, https://www.yandex.kz/search*, https://www.yandex.by/search*, https://www.yandex.uz/search*, https://www.yandex.com/search*, https://www.yandex.com.tr/search*, https://www.ya.ru/search*, https://yandex.ru/*, https://ya.ru/search*, https://yandex.kz/search*, https://yandex.by/search*, https://yandex.uz/search*, https://yandex.com/search*, https://yandex.com.tr/search*, https://ya.ru/search*


</code>

Код вставляем в поле с JavaScript, сохраняем, наслаждаемся


Сам JS-код

```

(function () {
  'use strict';

  // Нормализация доменов
  function toASCII(domain) {
    try {
      return new URL('http://' + domain).hostname.toLowerCase();
    } catch {
      return domain.toLowerCase();
    }
  }

  // Список ваших доменов
  const myDomains = ['mydomain1.ru', 'mydomain2.com', 'mydomain3.pro'].map(toASCII);

  // Список доменов для проверки рекламы
  const adDomains = [
    'https://yandex.ru/search/',
    'https://ya.ru/search/',
    'https://yandex.kz/search/',
    'https://yandex.by/search/',
    'https://yandex.com/search/',
    'https://yandex.uz/search/',
    'https://yandex.com.tr/search/',
    'https://yandex.ua/search/',
    'https://yabs.yandex.ru/count/',
    'https://yabs.ya.ru/count/',
    'https://yabs.yandex.kz/count/',
    'https://yabs.yandex.by/count/',
    'https://yabs.yandex.uz/count/',
    'https://yabs.yandex.com.tr/count/',
    'https://www.yandex.ru/search/',
    'https://www.ya.ru/search/',
    'https://www.yandex.kz/search/',
    'https://www.yandex.by/search/',
    'https://www.yandex.com/search/',
    'https://www.yandex.uz/search/',
    'https://www.yandex.com.tr/search/',
    'https://www.yandex.ua/search/',
    'https://www.yabs.yandex.ru/count/',
    'https://www.yabs.ya.ru/count/',
    'https://www.yabs.yandex.kz/count/',
    'https://www.yabs.yandex.by/count/',
    'https://www.yabs.yandex.uz/count/',
    'https://www.yabs.yandex.com.tr/count/'
  ];

  // Функция маркировки элементов
  function markAds(node) {
    if (node.nodeType !== Node.ELEMENT_NODE || node.dataset.marked === 'processed') return;

    // Проверка релевантных элементов
    const isSearchResultItem = (
      node.matches('#search-result > div, #search-result > li, #search-result > li > div') ||
      (node.classList.contains('serp-item') &&
        (node.classList.contains('serp-item_card') || node.classList.contains('serp-list__card'))) ||
      node.matches('[data-ajax-root="true"]') ||
      node.closest('[data-ajax-root="true"]')
    );

    if (isSearchResultItem) {
      // Проверка, находится ли элемент внутри div с классами complementary, content__right или любым data-fast-subtype
      const isExcluded = node.closest('.complementary, .content__right, [data-fast-subtype]');
      if (isExcluded) {
        node.dataset.marked = 'processed';
        return; // Пропускаем маркировку
      }

      let isAd = false;
      let hasMyDomain = false;

      // Проверка на рекламу
      const links = node.querySelectorAll('a:not(.MissingWords a)');
      for (const link of links) {
        if (adDomains.some(domain => link.href.startsWith(domain))) {
          isAd = true;
          break;
        }
      }

      // Дополнительная проверка на рекламу через метку
      if (!isAd && node.querySelector('.AdvLabel-Text')) {
        isAd = true;
      }

      // Проверка ваших доменов, если не реклама
      if (!isAd) {
        for (const link of links) {
          try {
            const url = new URL(link.href, window.location.origin);
            const hostname = url.hostname.toLowerCase();
            if (myDomains.some(domain => hostname === domain || hostname.endsWith('.' + domain))) {
              hasMyDomain = true;
              break;
            }
          } catch {
            // Пропускаем невалидные URL
          }
        }
      }

      // Применение цвета
      const color = isAd ? '#B79900' : hasMyDomain ? '#009159' : '';
      if (color) {
        node.style.backgroundColor = color;
      }

      // Обработка вложенных PromoOffer
      node.querySelectorAll('.PromoOffer').forEach((offer) => {
        if (color) offer.style.backgroundColor = color;
        offer.dataset.marked = 'processed';
      });

      node.dataset.marked = 'processed';
    }

    // Обработка корневых PromoOffer
    if (node.classList.contains('PromoOffer') && node.dataset.marked !== 'processed') {
      const parentNode = node.closest(
        '#search-result > div, #search-result > li, #search-result > li > div, .serp-item.serp-item_card, .serp-item.serp-list__card, [data-ajax-root="true"]'
      );
      if (parentNode) {
        // Проверка исключений для PromoOffer
        const isExcluded = parentNode.closest('.complementary, .content__right, [data-fast-subtype]');
        if (!isExcluded) {
          const bg = getComputedStyle(parentNode).backgroundColor;
          if (bg === 'rgb(183, 153, 0)') {
            node.style.backgroundColor = '#B79900';
          } else if (bg === 'rgb(0, 145, 89)') {
            node.style.backgroundColor = '#009159';
          }
        }
        node.dataset.marked = 'processed';
      }
    }
  }

  // Функция для маркировки всех релевантных элементов
  function markAll() {
    const selector = '#search-result > div, #search-result > li, #search-result > li > div, .serp-item.serp-item_card, .serp-item.serp-list__card, [data-ajax-root="true"], .PromoOffer';
    document.querySelectorAll(selector).forEach(markAds);
  }

  // Настройка MutationObserver
  function setupObserver() {
    const container = document.querySelector('#search-result') || document.body;
    const observer = new MutationObserver((mutations) => {
      mutations.forEach(mutation => {
        if (mutation.type === 'childList') {
          mutation.addedNodes.forEach(node => {
            if (node.nodeType === Node.ELEMENT_NODE) {
              markAds(node);
              node.querySelectorAll(
                '#search-result > div, #search-result > li, #search-result > li > div, .serp-item.serp-item_card, .serp-item.serp-list__card, [data-ajax-root="true"], .PromoOffer'
              ).forEach(markAds);
            }
          });
        }
      });
    });
    observer.observe(container, { childList: true, subtree: true });
    return observer;
  }

  // Периодическая проверка для AJAX-подгрузки
  function triggerCheck() {
    let count = 0;
    const maxChecks = 10; // 5 секунд
    const interval = setInterval(() => {
      markAll();
      if (++count >= maxChecks) clearInterval(interval);
    }, 500);
  }

  // Перехват AJAX-запросов
  function hookAjax() {
    const originalFetch = window.fetch;
    window.fetch = function () {
      const promise = originalFetch.apply(this, arguments);
      promise.then(() => setTimeout(triggerCheck, 100));
      return promise;
    };

    const originalOpen = XMLHttpRequest.prototype.open;
    XMLHttpRequest.prototype.open = function () {
      this.addEventListener('load', () => setTimeout(triggerCheck, 100));
      return originalOpen.apply(this, arguments);
    };

    if (window.jQuery) {
      const originalAjax = jQuery.ajax;
      jQuery.ajax = function () {
        const promise = originalAjax.apply(this, arguments);
        promise.done(() => setTimeout(triggerCheck, 100));
        return promise;
      };
    }
  }

  // Обработка событий
  function setupEventListeners() {
    document.addEventListener('click', (event) => {
      if (event.target.closest('a, button, [role="button"], .serp-item, .serp-item_card, .serp-list__card, [data-ajax-trigger], [data-load-more]')) {
        triggerCheck();
      }
    });

    document.addEventListener('scroll', () => {
      if (window.scrollY + window.innerHeight >= document.documentElement.scrollHeight - 100) {
        triggerCheck();
      }
    });
  }

  // Запуск скрипта
  function start() {
    const observer = setupObserver();
    markAll();
    hookAjax();
    setupEventListeners();

    window.stopObserving = function () {
      observer.disconnect();
    };
  }

  // Ожидание загрузки #search-result
  const waitInterval = setInterval(() => {
    if (document.querySelector('#search-result')) {
      clearInterval(waitInterval);
      start();
    }
  }, 300);

  // Таймаут на случай, если #search-result не появится
  setTimeout(() => {
    clearInterval(waitInterval);
    start();
  }, 5000);
})();
