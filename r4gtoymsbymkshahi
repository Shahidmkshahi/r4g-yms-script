// ==UserScript==
// @name         R4G to YMS Search with Site Selection
// @namespace    http://tampermonkey.net/
// @version      2.7
// @description  Adds YMS text links for redirection with auto-filled license plate or trailer ID, yard selection.
// @author       Shahid
// @match        https://trans-logistics.amazon.com/yms/sesameGateConsole*
// @match        https://trans-logistics.amazon.com/yms/shipclerk/*
// @grant        GM_addStyle
// ==/UserScript==

(function() {
    'use strict';

    GM_addStyle(`
        .ymsLink {
            margin-left: 5px;
            color: #0066c0;
            cursor: pointer;
            font-size: 12px;
            text-decoration: underline;
            display: inline-block;
        }
        .ymsLink:hover {
            color: #c45500;
        }
    `);

    function getSiteCode() {
        const siteCodeElement = document.querySelector('[data-testid="operationBuildingCodePurposeText"]');
        return siteCodeElement
            ? (siteCodeElement.getAttribute('mdn-text') || siteCodeElement.textContent.trim()).split(' ')[0]
            : null;
    }

    function handleYMSSearch() {
        const maxAttempts = 20;
        let attempts = 0;

        const searchInterval = setInterval(() => {
            attempts++;

            if (attempts >= maxAttempts) {
                clearInterval(searchInterval);
                return;
            }

            const searchInput = document.querySelector('#searchInput');
            const goToYardButton = document.querySelector('#goToYard');

            if (searchInput && goToYardButton) {
                clearInterval(searchInterval);

                const hashParams = new URLSearchParams(window.location.hash.split('?')[1] || '');
                const searchValue = hashParams.get('searchValue');
                const siteCode = hashParams.get('siteCode');

                if (searchValue && siteCode) {
                    const currentYard = goToYardButton.textContent.trim().split(' ').pop();

                    if (currentYard !== siteCode) {
                        goToYardButton.click();
                        setTimeout(() => {
                            performSearch(searchInput, searchValue);
                        }, 1500);
                    } else {
                        performSearch(searchInput, searchValue);
                    }
                }
            }
        }, 500);
    }

    function performSearch(searchInput, searchValue) {
        // Uncheck filters
        const filters = ['slip', 'empty'].forEach(filterName => {
            const checkbox = document.querySelector(`input[ng-model="topbar.filters.${filterName}"]`);
            if (checkbox && checkbox.checked) {
                const scope = angular.element(checkbox).scope();
                if (scope) {
                    scope.$apply(() => {
                        checkbox.checked = false;
                        checkbox.dispatchEvent(new Event('change'));
                    });
                }
            }
        });

        // Set search value and trigger search
        const scope = angular.element(searchInput).scope();
        if (scope) {
            scope.$apply(() => {
                // Update the input value
                searchInput.value = searchValue;

                // Update the Angular model
                scope.topbar.filters.searchQuery = searchValue;

                // Manually trigger input event
                searchInput.dispatchEvent(new Event('input', { bubbles: true }));

                // If there's a search function, call it
                if (typeof scope.topbar.textSearch === 'function') {
                    scope.topbar.textSearch(searchValue);
                }
            });
        }
    }

    function addYMSLinks() {
        const siteCode = getSiteCode();
        if (!siteCode) return;

        const createLink = (value) => {
            if (!value || value === '---') return null;

            const link = document.createElement('span');
            link.className = 'ymsLink';
            link.textContent = 'YMS';
            link.title = 'Open in YMS Search';
            link.addEventListener('click', () => {
                const params = new URLSearchParams({
                    locationType: 'ParkingLocation',
                    yardAssetStatus: 'EMPTY',
                    searchValue: value,
                    siteCode: siteCode
                });
                window.open(`https://trans-logistics.amazon.com/yms/shipclerk/#/yard?${params}`, '_blank');
            });
            return link;
        };

        // Add links to various containers
        [
            {
                selector: 'span[data-testid="plateContainer"]',
                valueSelector: 'div[data-testid="licensePlateNumber"]'
            },
            {
                selector: 'span[data-testid="trailerIdContainer"]',
                valueSelector: 'div[data-testid="trailerId"]'
            },
            {
                selector: 'div[data-testid="ownerCode"]'
            },
            {
                selector: '#app > div > div.css-1pu96e2 > div.css-m3vumr > div.css-xubts > div > div.r4g-drop-container > div > ul > li:nth-child(3) > div > span:nth-child(3) > div:nth-child(2)'
            }
        ].forEach(({selector, valueSelector}) => {
            document.querySelectorAll(selector).forEach(container => {
                if (container.querySelector('.ymsLink')) return;

                const value = valueSelector
                    ? container.querySelector(valueSelector)?.textContent.trim()
                    : container.textContent.trim();

                const link = createLink(value);
                if (link) container.appendChild(link);
            });
        });
    }

    // Initialize based on page
    if (window.location.href.includes('/yms/shipclerk/')) {
        handleYMSSearch();
    } else {
        const observer = new MutationObserver(() => addYMSLinks());
        observer.observe(document.body, { childList: true, subtree: true });
        addYMSLinks();
    }
})();
