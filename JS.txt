
// Toggle mode - Night mode Start
document.addEventListener('DOMContentLoaded', function() {
    const darkModeToggle = document.getElementById('dark-mode-toggle');
    const body = document.body;
    
    // Check for saved user preference
    if (localStorage.getItem('darkMode') === 'enabled') {
        body.classList.add('dark-mode');
        darkModeToggle.checked = true;
    }
    
    // Toggle dark mode
    darkModeToggle.addEventListener('change', function() {
        if (this.checked) {
            body.classList.add('dark-mode');
            localStorage.setItem('darkMode', 'enabled');
        } else {
            body.classList.remove('dark-mode');
            localStorage.setItem('darkMode', 'disabled');
        }
    });
});
// Toggle mode - Night mode end 

document.addEventListener('DOMContentLoaded', function() {
    // ========== Timeline Section ==========
    const timelineContainer = document.querySelector('.timeline-horizontal');
    const progressBar = document.getElementById('progressBar');
    const timelineCards = document.querySelectorAll('.timeline-card');
    const scrollLeftBtn = document.querySelector('.scroll-btn.left');
    const scrollRightBtn = document.querySelector('.scroll-btn.right');

    // Timeline Section Start scroll function
 // ? FIXED TIMELINE SCRIPT TO PREVENT PAGE JUMP ON LOAD
// This version avoids auto-scrolling to the timeline section on refresh unless it's already visible

window.addEventListener("DOMContentLoaded", () => {
  const timelineSection = document.querySelector('.timeline-section');
  if (!timelineSection) {
    console.error('Timeline section not found. Script cannot initialize.');
    return;
  }

  const nodes = timelineSection.querySelectorAll('.time-node');
  const cards = timelineSection.querySelectorAll('.event-card');
  const timelineCards = timelineSection.querySelector('#timelineCards');

  function isElementInViewport(el) {
    const rect = el.getBoundingClientRect();
    return (
      rect.top >= 0 &&
      rect.left >= 0 &&
      rect.bottom <= (window.innerHeight || document.documentElement.clientHeight) &&
      rect.right <= (window.innerWidth || document.documentElement.clientWidth)
    );
  }

  function activateTimeline(idx, smooth = true, onlyIfInView = false) {
    nodes.forEach((btn, i) => {
      btn.classList.toggle('active', i === idx);
    });
    cards.forEach((card, i) => {
      card.classList.toggle('active', i === idx);
    });

    setTimeout(() => {
      const activeNode = nodes[idx];
      const activeCard = cards[idx];

      if (activeCard && activeNode) {
        if (onlyIfInView && !isElementInViewport(timelineSection)) return;

        if (window.innerWidth >= 768) {
          activeCard.scrollIntoView({
            behavior: smooth ? "smooth" : "auto",
            block: "center",
            inline: "center"
          });
        } else {
          activeCard.scrollIntoView({
            behavior: smooth ? "smooth" : "auto",
            block: "center"
          });
        }
      }
    }, 30);
  }

  nodes.forEach((btn, idx) => {
    btn.addEventListener('click', () => {
      activateTimeline(idx);
    });
    btn.addEventListener('keydown', e => {
      if (e.key === ' ' || e.key === 'Enter') {
        e.preventDefault();
        activateTimeline(idx);
      }
      const isDesktop = window.innerWidth >= 768;
      if ((isDesktop && e.key === 'ArrowRight') || (!isDesktop && e.key === 'ArrowDown')) {
        if (nodes[idx + 1]) {
          nodes[idx + 1].focus();
          activateTimeline(idx + 1);
        }
      }
      if ((isDesktop && e.key === 'ArrowLeft') || (!isDesktop && e.key === 'ArrowUp')) {
        if (nodes[idx - 1]) {
          nodes[idx - 1].focus();
          activateTimeline(idx - 1);
        }
      }
    });
  });

  cards.forEach((card, idx) => {
    card.addEventListener('click', () => activateTimeline(idx));
    card.addEventListener('keydown', e => {
      if (e.key === ' ' || e.key === 'Enter') {
        e.preventDefault();
        activateTimeline(idx);
      }
    });
  });

  const observerOptions = {
    root: null,
    rootMargin: '0px',
    threshold: 0.2
  };
  const observer = new IntersectionObserver((entries, observer) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        entry.target.classList.add('is-visible');
        observer.unobserve(entry.target);
      }
    });
  }, observerOptions);

  cards.forEach(card => {
    observer.observe(card);
  });

  function handleResize() {
    const activeNode = timelineSection.querySelector('.time-node.active');
    if (activeNode) {
      const idx = Array.from(nodes).indexOf(activeNode);
      activateTimeline(idx, false, true);
    }
  }
  window.addEventListener('resize', handleResize);

  // ? Only activate the first card if timeline is already in view
  if (isElementInViewport(timelineSection)) {
    activateTimeline(0, false);
  }
});


  // Timeline Section END scroll function
   

    // Connection lines between cards
    function addConnectionLines() {
        if (window.innerWidth >= 768 && timelineContainer) {
            const cardsToConnect = document.querySelectorAll('.timeline-card:not(.future)');
            document.querySelectorAll('.connection-line').forEach(line => line.remove());

            const fragment = document.createDocumentFragment();
            
            for (let i = 0; i < cardsToConnect.length - 1; i++) {
                const currentCard = cardsToConnect[i];
                const nextCard = cardsToConnect[i + 1];
                
                if (!nextCard || !currentCard.offsetParent || !nextCard.offsetParent) continue;

                const line = document.createElement('div');
                line.className = 'connection-line';
                
                const startX = currentCard.offsetLeft + currentCard.offsetWidth;
                const endX = nextCard.offsetLeft;
                const y = currentCard.offsetTop + currentCard.offsetHeight / 2;

                line.style.left = `${startX}px`;
                line.style.width = `${endX - startX}px`;
                line.style.top = `${y}px`;
                
                fragment.appendChild(line);
            }
            
            timelineContainer.appendChild(fragment);
        }
    }

    // Add connection line styles
    if (!document.querySelector('style[data-connection-lines]')) {
        const style = document.createElement('style');
        style.setAttribute('data-connection-lines', 'true');
        style.textContent = `
            .connection-line {
                position: absolute;
                height: 2px;
                background: linear-gradient(to right, rgba(248, 181, 0, 0.5), rgba(248, 181, 0, 0.2));
                z-index: 1;
                pointer-events: none;
                transition: all 0.3s ease;
            }
        `;
        document.head.appendChild(style);
    }

    // Initialize connection lines
    addConnectionLines();
    window.addEventListener('resize', () => {
        clearTimeout(window._resizeDebounce);
        window._resizeDebounce = setTimeout(addConnectionLines, 200);
    });

    // ========== Stats Counter Animation ==========
    function animateValue(element, start, end, duration) {
        let startTimestamp = null;
        const step = (timestamp) => {
            if (!startTimestamp) startTimestamp = timestamp;
            const progress = Math.min((timestamp - startTimestamp) / duration, 1);
            element.textContent = Math.floor(progress * (end - start) + start) + (end >= 1000 ? '+' : '');
            if (progress < 1) window.requestAnimationFrame(step);
        };
        window.requestAnimationFrame(step);
    }

    function handleScrollAnimation() {
        const stats = document.querySelectorAll('.stat-card');
        if (!stats.length) return;

        const statObserver = new IntersectionObserver((entries, obs) => {
            entries.forEach(entry => {
                if (entry.isIntersecting) {
                    const card = entry.target;
                    const target = +card.getAttribute('data-target');
                    const numberElem = card.querySelector('.stat-number');
                    if (numberElem) {
                        animateValue(numberElem, 0, target, 1500);
                    }
                    obs.unobserve(card);
                }
            });
        }, { threshold: 0.6 });

        stats.forEach(stat => statObserver.observe(stat));
    }
    handleScrollAnimation();
  
  

    // ========== 3D Carousel ==========
    const carouselContainer = document.getElementById('carouselContainer');
    if (carouselContainer) {
        const allCardWrappers = document.querySelectorAll('.card-wrapper');
        let cardWrappers = [];
        const themeButtons = document.querySelectorAll('.themes span');
        const navDots = document.getElementById('navDots');
        const prevBtn = document.getElementById('prevBtn');
        const nextBtn = document.getElementById('nextBtn');
        const viewButtons = document.querySelectorAll('.view-btn');

        let activeCardIndex = 0;
        let touchStartX = 0;
        let touchEndX = 0;
        let isDragging = false;
        let dragStartX = 0;
        let dragOffset = 0;
        let dots = [];

        function filterCards(theme) {
            cardWrappers = [];
            allCardWrappers.forEach(wrapper => {
                const cardTheme = wrapper.getAttribute('data-theme');
                if (theme === 'all' || cardTheme === theme) {
                    wrapper.classList.remove('hidden');
                    cardWrappers.push(wrapper);
                } else {
                    wrapper.classList.add('hidden');
                }
            });
            activeCardIndex = 0;
            updateCardPositions();
            createDots();
        }

        function createDots() {
            if (!navDots) return;
            navDots.innerHTML = '';
            dots = [];
            
            if (cardWrappers.length <= 1) {
                navDots.style.display = 'none';
                return;
            }
            
            navDots.style.display = 'flex';
            cardWrappers.forEach((_, index) => {
                const dot = document.createElement('div');
                dot.classList.add('dot');
                if (index === activeCardIndex) dot.classList.add('active');
                dot.addEventListener('click', () => {
                    activeCardIndex = index;
                    updateCardPositions();
                    updateDots();
                });
                navDots.appendChild(dot);
                dots.push(dot);
            });
        }

        function updateDots() {
            dots.forEach((dot, index) => {
                dot.classList.toggle('active', index === activeCardIndex);
            });
        }

        function handleTouchStart(e) {
            isDragging = true;
            carouselContainer.classList.add('dragging');
            touchStartX = e.type === 'touchstart' ? e.touches[0].clientX : e.clientX;
            dragStartX = touchStartX;
            dragOffset = 0;

            cardWrappers.forEach(wrapper => {
                wrapper.style.transition = 'none';
            });
            
            if (e.type === 'mousedown') {
                e.preventDefault();
            }
        }

        function handleTouchMove(e) {
            if (!isDragging) return;
            const touchX = e.type === 'touchmove' ? e.touches[0].clientX : e.clientX;
            dragOffset = touchX - dragStartX;
            updateCardPositionsDuringDrag();
        }

        function handleTouchEnd(e) {
            if (!isDragging) return;
            isDragging = false;
            carouselContainer.classList.remove('dragging');

            cardWrappers.forEach(wrapper => {
                wrapper.style.transition = 'all 0.6s cubic-bezier(0.22, 1, 0.36, 1)';
            });

            touchEndX = e.type === 'touchend' ? e.changedTouches[0].clientX : e.clientX;
            handleSwipe();
        }

        function updateCardPositionsDuringDrag() {
            const isMobile = window.innerWidth <= 768;

            cardWrappers.forEach((wrapper, index) => {
                const diff = index - activeCardIndex;
                let translateX, scale, rotateY, opacity, zIndex;

                if (diff === 0) {
                    translateX = dragOffset;
                    scale = 1.1 - (Math.abs(dragOffset) / (isMobile ? 300 : 400) * 0.2);
                    rotateY = 0 + (dragOffset / (isMobile ? 15 : 25));
                    opacity = 1;
                    zIndex = 10;
                } else if (Math.abs(diff) === 1) {
                    translateX = diff * (isMobile ? 160 : 220) + dragOffset * 0.7;
                    scale = (isMobile ? 0.85 : 0.8) + (Math.abs(dragOffset) / (isMobile ? 300 : 400) * 0.1);
                    rotateY = diff * (isMobile ? 15 : 25) + (dragOffset / (isMobile ? 15 : 25));
                    opacity = 0.8;
                    zIndex = 2;
                } else if (Math.abs(diff) === 2) {
                    translateX = diff * (isMobile ? 300 : 420) + dragOffset * 0.5;
                    scale = (isMobile ? 0.75 : 0.7);
                    rotateY = diff * (isMobile ? 30 : 45);
                    opacity = isMobile ? 0.3 : 0;
                    zIndex = 1;
                } else {
                    translateX = diff * (isMobile ? 420 : 600) + dragOffset * 0.3;
                    scale = 0.6;
                    rotateY = diff * (isMobile ? 45 : 60);
                    opacity = 0;
                    zIndex = 0;
                }

                wrapper.style.transform = `translateX(${translateX}px) scale(${scale}) rotateY(${rotateY}deg)`;
                wrapper.style.opacity = opacity;
                wrapper.style.zIndex = Math.round(zIndex);
            });
        }

        function handleSwipe() {
            const swipeThreshold = 50;
            const swipeDistance = touchEndX - touchStartX;

            if (Math.abs(swipeDistance) > swipeThreshold) {
                navigate(swipeDistance < 0 ? 1 : -1);
            } else {
                updateCardPositions();
            }
        }

        function navigate(direction) {
            if (cardWrappers.length === 0) return;
            activeCardIndex = (activeCardIndex + direction + cardWrappers.length) % cardWrappers.length;
            updateCardPositions();
            updateDots();
        }

        function updateCardPositions() {
            const isMobile = window.innerWidth <= 768;

            cardWrappers.forEach((wrapper, index) => {
                const diff = index - activeCardIndex;
                let translateX, scale, rotateY, opacity, zIndex;

                if (diff === 0) {
                    translateX = 0;
                    scale = 1.1;
                    rotateY = 0;
                    opacity = 1;
                    zIndex = 10;
                    wrapper.classList.add('active');
                } else {
                    wrapper.classList.remove('active');
                    if (Math.abs(diff) === 1) {
                        translateX = diff * (isMobile ? 160 : 220);
                        scale = isMobile ? 0.85 : 0.8;
                        rotateY = diff * (isMobile ? 15 : 25);
                        opacity = 0.8;
                        zIndex = 2;
                    } else if (Math.abs(diff) === 2) {
                        translateX = diff * (isMobile ? 300 : 420);
                        scale = isMobile ? 0.75 : 0.7;
                        rotateY = diff * (isMobile ? 30 : 45);
                        opacity = isMobile ? 0.3 : 0;
                        zIndex = 1;
                    } else {
                        translateX = diff * (isMobile ? 420 : 600);
                        scale = 0.6;
                        rotateY = diff * (isMobile ? 45 : 60);
                        opacity = 0;
                        zIndex = 0;
                    }
                }

                wrapper.style.transform = `translateX(${translateX}px) scale(${scale}) rotateY(${rotateY}deg)`;
                wrapper.style.opacity = opacity;
                wrapper.style.zIndex = zIndex;
            });

            if (prevBtn) prevBtn.style.display = cardWrappers.length > 1 ? 'flex' : 'none';
            if (nextBtn) nextBtn.style.display = cardWrappers.length > 1 ? 'flex' : 'none';
        }

        // Event listeners
        if (themeButtons.length) {
            themeButtons.forEach(button => {
                button.addEventListener('click', () => {
                    themeButtons.forEach(btn => btn.classList.remove('active'));
                    button.classList.add('active');
                    filterCards(button.getAttribute('data-theme'));
                });
            });
        }

        if (viewButtons.length) {
            viewButtons.forEach(button => {
                button.addEventListener('click', (e) => {
                    e.stopPropagation();
                    console.log('View gallery button clicked!');
                });
            });
        }

        if (prevBtn) prevBtn.addEventListener('click', () => navigate(-1));
        if (nextBtn) nextBtn.addEventListener('click', () => navigate(1));

        document.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowLeft') navigate(-1);
            else if (e.key === 'ArrowRight') navigate(1);
        });

        carouselContainer.addEventListener('mousedown', handleTouchStart);
        carouselContainer.addEventListener('mousemove', handleTouchMove);
        carouselContainer.addEventListener('mouseup', handleTouchEnd);
        carouselContainer.addEventListener('mouseleave', handleTouchEnd);
        carouselContainer.addEventListener('touchstart', handleTouchStart, { passive: false });
        carouselContainer.addEventListener('touchmove', handleTouchMove, { passive: false });
        carouselContainer.addEventListener('touchend', handleTouchEnd, { passive: false });

        // Initialize
        filterCards('all');
        window.addEventListener('resize', updateCardPositions);
    }
});


// Bento Style Wedding Image Grid Section with Lightbox Start
// Lightbox Functionality
document.querySelectorAll('.card').forEach(card => {
    card.addEventListener('click', function() {
        const lightbox = document.getElementById('lightbox');
        const lightboxImg = document.getElementById('lightbox-img');
        lightboxImg.src = this.getAttribute('data-image');
        lightbox.classList.add('active');
        document.body.style.overflow = 'hidden';
    });
});

function closeLightbox() {
    const lightbox = document.getElementById('lightbox');
    lightbox.classList.remove('active');
    document.body.style.overflow = '';
}

// Close lightbox when clicking outside image
document.getElementById('lightbox').addEventListener('click', function(e) {
    if (e.target === this) {
        closeLightbox();
    }
});

// Dark Mode Toggle
document.getElementById('darkModeToggle').addEventListener('click', function() {
    document.body.classList.toggle('dark-mode');
    this.textContent = document.body.classList.contains('dark-mode') ? '??' : '??';
});

// Keyboard Navigation
document.addEventListener('keydown', function(e) {
    const lightbox = document.getElementById('lightbox');
    if (e.key === 'Escape' && lightbox.classList.contains('active')) {
        closeLightbox();
    }
});

// Bento Style Wedding Image Grid Section with Lightbox END


// Proffessional Equipment bento style start
 // Enhanced Modal functionality
        document.querySelectorAll('.read-more-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                e.stopPropagation();
                const equipmentItem = btn.closest('.equipment-item');
                const title = equipmentItem.getAttribute('data-title');
                const brand = equipmentItem.getAttribute('data-brand');
                const description = equipmentItem.getAttribute('data-description');
                const features = equipmentItem.getAttribute('data-features');
                
                // Update modal content
                document.getElementById('modalTitle').textContent = title;
                document.getElementById('modalBrand').textContent = brand;
                document.getElementById('modalDescription').textContent = description;
                
                // Create feature tags
                const featuresContainer = document.getElementById('modalFeatures');
                featuresContainer.innerHTML = '';
                
                if (features) {
                    features.split(',').forEach(feature => {
                        const tag = document.createElement('span');
                        tag.className = 'feature-tag';
                        tag.textContent = feature.trim();
                        featuresContainer.appendChild(tag);
                    });
                }
                
                document.getElementById('equipmentModal').style.display = 'flex';
            });
        });

        document.querySelector('.modal-close').addEventListener('click', () => {
            document.getElementById('equipmentModal').style.display = 'none';
        });

        window.addEventListener('click', (e) => {
            const modal = document.getElementById('equipmentModal');
            if (e.target === modal) {
                modal.style.display = 'none';
            }
        });

        // Keyboard support for accessibility
        document.addEventListener('keydown', (e) => {
            if (e.key === 'Escape') {
                document.getElementById('equipmentModal').style.display = 'none';
            }
        });


// Proffessional Equipment Bento style end


//3D SLIDER IMAGE CURSOEL STYLE END

(function () {
  const members = [
    { name: "Pre-Wedding Shoot", role: "Capture the chemistry, candid moments, and love story before the big day in scenic or personalized locations." },
    { name: "Engagement Shoot", role: "Document the joy and excitement of your engagement in a stylish and memorable session." },
    { name: "Haldi Ceremony", role: "Bright, colorful, and full of energy—our lens catches every splash of turmeric and laughter." },
    { name: "Sangeet & Musical Night", role: "High-energy coverage of dances, fun, and family performances in cinematic style." },
    { name: "Drone Wedding Photography", role: "Aerial shots of venues, baraat, and ceremonies from stunning perspectives." },
    { name: "Bridal Portraits", role: "Elegantly styled portraits capturing the bride's grace, beauty, and attire details." }
  ];

  const cards = document.querySelectorAll("#wedding-carousel-unique .wedding-carousel-card");
  const dots = document.querySelectorAll("#wedding-carousel-unique .wedding-carousel-dot");
  const nameEl = document.querySelector("#wedding-carousel-unique .wedding-carousel-name");
  const roleEl = document.querySelector("#wedding-carousel-unique .wedding-carousel-description");
  const leftBtn = document.querySelector("#wedding-carousel-unique .wedding-carousel-nav-left");
  const rightBtn = document.querySelector("#wedding-carousel-unique .wedding-carousel-nav-right");

  let index = 0;
  let isMoving = false;

  function updateCarousel(newIndex) {
    if (isMoving) return;
    isMoving = true;
    index = (newIndex + cards.length) % cards.length;

    cards.forEach((card, i) => {
      card.classList.remove("center", "left-1", "left-2", "right-1", "right-2", "hidden");
      const offset = (i - index + cards.length) % cards.length;
      if (offset === 0) card.classList.add("center");
      else if (offset === 1) card.classList.add("right-1");
      else if (offset === 2) card.classList.add("right-2");
      else if (offset === cards.length - 1) card.classList.add("left-1");
      else if (offset === cards.length - 2) card.classList.add("left-2");
      else card.classList.add("hidden");
    });

    dots.forEach((dot, i) => {
      dot.classList.toggle("active", i === index);
    });

    nameEl.style.opacity = "0";
    roleEl.style.opacity = "0";
    setTimeout(() => {
      nameEl.textContent = members[index].name;
      roleEl.textContent = members[index].role;
      nameEl.style.opacity = "1";
      roleEl.style.opacity = "1";
    }, 300);

    setTimeout(() => isMoving = false, 800);
  }

  leftBtn.addEventListener("click", () => updateCarousel(index - 1));
  rightBtn.addEventListener("click", () => updateCarousel(index + 1));
  dots.forEach((dot, i) => dot.addEventListener("click", () => updateCarousel(i)));
  cards.forEach((card, i) => card.addEventListener("click", () => updateCarousel(i)));
  document.addEventListener("keydown", e => {
    if (e.key === "ArrowLeft") updateCarousel(index - 1);
    else if (e.key === "ArrowRight") updateCarousel(index + 1);
  });
  document.addEventListener("touchstart", e => touchStartX = e.changedTouches[0].screenX);
  document.addEventListener("touchend", e => {
    const diff = touchStartX - e.changedTouches[0].screenX;
    if (Math.abs(diff) > 50) updateCarousel(index + (diff > 0 ? 1 : -1));
  });

  let touchStartX = 0;
  updateCarousel(0);
})();

//3D SLIDER IMAGE CURSOEL STYLE END

//Contact form start
document.addEventListener('DOMContentLoaded', () => {
    const form = document.getElementById('contactForm');
    const nameInput = document.getElementById('name');
    const emailInput = document.getElementById('email');
    const messageInput = document.getElementById('message');
    const submitBtn = document.querySelector('.submit-btn25'); // Updated class name
    
    // Custom Message Box Elements
    const messageOverlay = document.getElementById('messageOverlay');
    const messageBox = document.getElementById('messageBox');
    const messageIcon = document.getElementById('messageIcon');
    const messageTitle = document.getElementById('messageTitle');
    const messageText = document.getElementById('messageText');
    const closeMessageBtn = document.getElementById('closeMessageBtn');

    // Function to show custom message box
    function showMessageBox(title, text, type) {
        messageTitle.textContent = title;
        messageText.textContent = text;
        messageBox.classList.remove('success', 'error');
        if (type === 'success') {
            messageBox.classList.add('success');
            messageIcon.className = 'fas fa-check-circle';
        } else if (type === 'error') {
            messageBox.classList.add('error');
            messageIcon.className = 'fas fa-exclamation-circle';
        }
        messageOverlay.style.display = 'flex';
        setTimeout(() => messageOverlay.classList.add('show'), 10);
    }

    // Function to hide custom message box
    function hideMessageBox() {
        messageOverlay.classList.remove('show');
        setTimeout(() => {
            messageOverlay.style.display = 'none';
        }, 300); // Wait for the transition to finish
    }
    
    closeMessageBtn.addEventListener('click', hideMessageBox);

    // Form validation
    function validateForm() {
        const name = nameInput.value.trim();
        const email = emailInput.value.trim();
        const message = messageInput.value.trim();
        
        if (!name || !email || !message) {
            showMessageBox('Validation Error', 'Please fill in all required fields.', 'error');
            return false;
        }
        
        if (!isValidEmail(email)) {
            showMessageBox('Validation Error', 'Please enter a valid email address.', 'error');
            return false;
        }
        
        return true;
    }

    // Email validation regex
    function isValidEmail(email) {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return emailRegex.test(email);
    }

    // Form submission logic
    form.addEventListener('submit', function(e) {
        e.preventDefault();
        
        if (!validateForm()) {
            return;
        }
        
        // Show loading state
        const originalText = submitBtn.innerHTML;
        submitBtn.innerHTML = '<i class="fas fa-spinner fa-spin"></i> Sending...';
        submitBtn.disabled = true;

        // Simulate form submission and open WhatsApp
        setTimeout(() => {
            createWhatsAppLink();
            
            showMessageBox('Success!', 'Thank you! Your message has been sent successfully.', 'success');
            
            form.reset();
            submitBtn.innerHTML = originalText;
            submitBtn.disabled = false;
        }, 1500);
    });
    
    // Create a WhatsApp link with form data
    function createWhatsAppLink() {
        const name = nameInput.value;
        const email = emailInput.value;
        const phone = document.getElementById('phone').value;
        const eventType = document.getElementById('event-type').value;
        const message = messageInput.value;

        const whatsappNumber = '7200552119'; // Replace with owner's full WhatsApp number (with country code, no +)

        const text = `
?? NEW INQUIRY - BILLION MEMORIES ??

?? Name: ${name}
?? Email: ${email}
?? Phone: ${phone}
?? Event Type: ${eventType || 'Not specified'}

?? Message:
${message}

?? Submitted on: ${new Date().toLocaleString()}
--------------------------
Website Contact Form Submission`;

        const url = `https://wa.me/${whatsappNumber}?text=${encodeURIComponent(text)}`;
        window.open(url, '_blank'); // Open WhatsApp in a new tab
    }

    // Typing animation for placeholder
    const messageTextarea = messageInput;
    const placeholderText = "Tell us about your event, preferred dates, and any special requirements...";
    let i = 0;
    let typingInterval;

    function typePlaceholder() {
        if (i < placeholderText.length) {
            messageTextarea.placeholder = placeholderText.slice(0, i + 1);
            i++;
        } else {
            clearInterval(typingInterval);
        }
    }

    messageTextarea.addEventListener('focus', () => {
        if (!typingInterval) {
            messageTextarea.placeholder = "";
        }
        clearInterval(typingInterval);
    });

    messageTextarea.addEventListener('blur', () => {
        if (!messageTextarea.value) {
            messageTextarea.placeholder = "";
            i = 0;
            typingInterval = setInterval(typePlaceholder, 50);
        }
    });
    
    // Initial placeholder animation
    typingInterval = setInterval(typePlaceholder, 50);

    // Intersection Observer for animations
    const observerOptions = {
        root: null,
        rootMargin: '0px',
        threshold: 0.1
    };

    const observer = new IntersectionObserver(function(entries) {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                entry.target.style.animationPlayState = 'running';
            }
        });
    }, observerOptions);

    document.querySelectorAll('.contact-item25').forEach((item, index) => { // Updated class name
        item.style.animationDelay = `${index * 0.2}s`;
        observer.observe(item);
    });
});

// Contact form END

//Foter 
document.addEventListener('DOMContentLoaded', () => {
    // Custom Message Box Elements for newsletter
    const newsletterMessageOverlay = document.getElementById('newsletterMessageOverlay');
    const newsletterMessageBox = document.getElementById('newsletterMessageBox');
    const newsletterMessageIcon = document.getElementById('newsletterMessageIcon');
    const newsletterMessageTitle = document.getElementById('newsletterMessageTitle');
    const newsletterMessageText = document.getElementById('newsletterMessageText');
    const closeNewsletterMessageBtn = document.getElementById('closeNewsletterMessageBtn');

    // Function to show custom message box for newsletter
    function showNewsletterMessageBox(title, text, type) {
        newsletterMessageTitle.textContent = title;
        newsletterMessageText.textContent = text;
        newsletterMessageBox.classList.remove('success', 'error');
        if (type === 'success') {
            newsletterMessageBox.classList.add('success');
            newsletterMessageIcon.className = 'fas fa-check-circle';
        } else if (type === 'error') {
            newsletterMessageBox.classList.add('error');
            newsletterMessageIcon.className = 'fas fa-exclamation-circle';
        }
        newsletterMessageOverlay.style.display = 'flex';
        setTimeout(() => newsletterMessageOverlay.classList.add('show'), 10);
    }

    // Function to hide custom message box for newsletter
    function hideNewsletterMessageBox() {
        newsletterMessageOverlay.classList.remove('show');
        setTimeout(() => {
            newsletterMessageOverlay.style.display = 'none';
        }, 300); // Wait for the transition to finish
    }
    
    closeNewsletterMessageBtn.addEventListener('click', hideNewsletterMessageBox);

    // Add smooth scrolling for back to top button
    const backToTopBtn = document.querySelector('.back-to-top25'); // Updated class name
    if (backToTopBtn) {
        backToTopBtn.addEventListener('click', function(e) {
            e.preventDefault();
            window.scrollTo({
                top: 0,
                behavior: 'smooth'
            });
        });
    }

    // Newsletter subscription (demo)
    const newsletterBtn = document.querySelector('.newsletter-btn25'); // Updated class name
    if (newsletterBtn) {
        newsletterBtn.addEventListener('click', function() {
            const emailInput = this.closest('.newsletter-input25').querySelector('input[type="email"]'); // Updated class name
            const email = emailInput.value.trim();
            if (email && isValidEmail(email)) {
                showNewsletterMessageBox('Subscription Successful!', 'Thank you for subscribing to our newsletter.', 'success');
                emailInput.value = ''; // Clear input field
            } else {
                showNewsletterMessageBox('Subscription Failed', 'Please enter a valid email address.', 'error');
            }
        });
    }

    // Email validation regex (reused from contact form)
    function isValidEmail(email) {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return emailRegex.test(email);
    }

    // Add hover effect to footer columns
    const footerColumns = document.querySelectorAll('.footer-column25'); // Updated class name
    footerColumns.forEach(column => {
        column.addEventListener('mouseenter', function() {
            this.style.transform = 'translateY(-5px)';
            this.style.transition = 'transform 0.3s ease';
        });
        
        column.addEventListener('mouseleave', function() {
            this.style.transform = 'translateY(0)';
        });
    });
});
