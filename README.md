// ==UserScript==
// @name         YouTube Playlist - Remove All Videos (Fixed & Faster)
// @namespace    http://tampermonkey.net/
// @version      1.8
// @description  Automatically removes ALL videos from your YouTube playlist continuously until empty.
// @author       Your Name
// @match        https://www.youtube.com/playlist?*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    function removeVideos() {
        const videoItems = document.querySelectorAll('ytd-playlist-video-renderer');

        if (videoItems.length === 0) {
            console.log("✅ No more videos to remove. Stopping script.");
            return; // Stop script when playlist is empty
        }

        let index = 0;

        function removeNextVideo() {
            if (index >= videoItems.length) {
                console.log("🔄 Finished one pass, checking for more videos...");
                setTimeout(removeVideos, 2000); // Check again in 2 seconds
                return;
            }

            const video = videoItems[index];
            const menuButton = video.querySelector('#button[aria-label="Action menu"]');

            if (!menuButton) {
                console.log(`❌ Menu button not found for video ${index + 1}, skipping...`);
                index++;
                removeNextVideo();
                return;
            }

            menuButton.click(); // Open the menu

            setTimeout(() => {
                const removeButton = [...document.querySelectorAll('ytd-menu-service-item-renderer')]
                    .find(button => button.innerText.includes("Remove from"));

                if (removeButton) {
                    removeButton.click(); // Click "Remove from playlist"
                    console.log(`✅ Removed video ${index + 1}`);
                    setTimeout(() => {
                        removeVideos(); // Restart function to handle next batch
                    }, 1000);
                } else {
                    console.log(`❌ Remove button not found for video ${index + 1}`);
                    index++;
                    setTimeout(removeNextVideo, 1000); // Try next video after 1 sec
                }
            }, 800); // Shorter wait for menu to open
        }

        removeNextVideo();
    }

    setTimeout(removeVideos, 2000); // Start script after 2 seconds for better page loading

})();
